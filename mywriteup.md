##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[calibration_test_0]: ./examples/test_calibration_0.jpg "Test calibration 0"
[calibration_test_1]: ./examples/test_calibration_1.jpg "Test calibration 1"

[gradients_0]: ./examples/gradients_0.jpg "Gradients try 0"
[combined_0]: ./examples/comined_without_s.jpg "Combinded without S chanel"
[hls_chanels]: ./examples/hls_chanels.jpg "HLS chanels"
[pipeline_result]: ./examples/result_pipeline.jpg "Pipeline result"
[warped_points]: ./examples/warped_points.jpg "Warped Points"
[warped]: ./examples/warped.jpg "Warped"

[detect_lines]: ./examples/detect_lines.jpg "Detecting lines"
[detected_lines]: ./examples/detected_lines.jpg "Detected lines"
[detected_lines_more_images]: ./examples/detected_lines_more_images.jpg "More detected lines"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./Advanced_Lines_Finder.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][output_2_0]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][calibration_test_1]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Gradients:
![alt text][gradients_0] 

As it is seen best result here were seen on Thresholded Gradient X image.

On the next step I decided to combine previous results using rule:
```
    combined[((grad_x_binary == 1) & (grad_y_binary == 1)) & ((mag_binary == 1) | (dir_binary == 1))] = 1
```
That results to:

![alt text][combined_0]

But as it is seen left yellow line was almost lost in this configuration. 

To restore yellow line I decided to take a look on S chanel after converting image to HLS colorspace:
 
![alt text][hls_chanels]


Combining thresholds for S chanel and results for previous thresholds processing I got the next frame: 
```
    combined_binary[(binary_s == 1) | (combined == 1)] = 1
```

![alt text][pipeline_result]

This result will be used later to find land lanes.

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
```
src = np.float32([[240, 686], 
                  [1070, 686], 
                  [738, 480], 
                  [545,  480]])
                      
dst = np.float32([[300, 700], 
                  [1000, 700], 
                  [1000, 300], 
                  [300, 300]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 240, 686      | 300, 700      | 
| 1070, 686     | 1000, 700     |
| 738, 480      | 1000, 300     |
| 545,  480     | 300, 300      |

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:


Input for this step was taken as a combined image after preprocessing pipeline and converted  to 'bird-eye' view using perspective transformation:

Then calculating a histogram and find out its peaks I found where lines are located. These points are treated as a starting ponits for lines detection algorithm.
On the next step image was separated by 9 windows and moving from the bottom to the top for each line. In each line was selected pixels that are not zero.
Each next window was recentered using mean position of non 0 pixels detected in previous step.

After pixels were detected polyfit function was used to find сoefficients for  second order polynomial.
```
# Fit a second order polynomial to each
line.current_fit = np.polyfit(line.ally, line.allx, 2)
```

Resulting polinomial image below.

![alt text][detect_lines]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `__calculate_radius_of_curvature()` and `__calculate_offest_from_center()` methods in cell 16 of my notebook.

![alt_text][detected_lines]

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][detected_lines_more_images]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4) or https://youtu.be/k2c5cSZ3qew


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

