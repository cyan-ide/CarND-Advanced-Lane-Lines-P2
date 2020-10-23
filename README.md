# **Advanced Lane Finding** 

## Udacity Project 2 Writeup

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

---

In comparison to course material the key code modificatoons / customizations are in:
* color transforms
* perspective transform
* pixel detection / line plotting
* sanity check

**Summary:** The solution was tested for single frames located in **"test_images"** directory, **project_video.mp4** and **challenge_video.mp4**. All should work in satisfactory degree, most of the frames being correctly annotated with lane. I made some quick tests for the harder_challenge_video.mp4, however I did not customize my solution to work for this scenario, therefore I do not include it.


[//]: # (Image References)

[calibration_images]: ./examples/writeup/example_calibration.jpg "Calibration Output"

[image1]: ./examples/writeup/undistored_image.jpg "Undistorted"
[image2]: ./examples/writeup/color_transform.jpg "Color Transform"
[image3]: ./examples/writeup/warped_image.png "Warped Image"
[image4]: ./examples/writeup/fitted_line.png "Fitted Line"
[image5]: ./examples/writeup/final_output.png "Final Output"


<!-- [image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video" -->

---

**Key files:**
* P2.ipynb - final code for the project
* README.md - (this file) - writeup on project coding, challenges and possible improvements

Additional files:
* original_README.md - original project readme file with the description of problem to solve

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell 'Preparation' of the IPython notebook located in "./P2.ipynb".  Function 'calibrateCammera'

Later on the function is executed as: mtx, dist = calibrateCammera(camera_calib_images, test = verbose_mode).

In this case, the function is almost exactly the same as the samples from the course exercises. I just modified the chessboard size to fit the calibration images (ie. 9x6).


An example of original image, detected corners and undistorted image can be see below.

![alt text][calibration_images]

### Pipeline (single images)

All the pipline steps are executed via **"extract_lane_boundries(image,mtx, dist, lane_data = LaneLines(), test = False)"** function. This function calls individual functions responsible for performing consecutive tasks in the line finding pipeline.

#### 1. [Distortion Correction] Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]

#### 2. [Image Transform] Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tested multiple transforms: sobel gradient x,y; HLS; RGB; LUV; LAB. For the still images using just sobel gradient in direction x already gave good results. For first video I had to add S channel from HLS. Finally, for the challenge video it became apparent that something more was needed. That's when I experimented with remaining ones, in the end settling for R+G from RGB and L from HLS.

Below can be seen samples of all transforms applied to image and final joint image.

![alt text][image2]

#### 3. [Warping] Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Similarly as during the class examples I hardcoded the source and destination points as below.

| Source        | Destination   	| 
|:-------------:|:-------------:	| 
| 150, 720      | 250,  720 	 	| 
| 570, 460      | 250,    0      	|
| 710, 460     	| 1065,   0      	|
| 1175, 720     | 1065, 720      	|

The original with plotted source coordinates and warped images with destination coordinates can be seen below.

![alt text][image3]

#### 4. [Line Fitting] Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For line fitting I started with functions coded in class and I customised their execution with some logic that selects which function to use depending on the sanity check of fitted line. The logic to switch between histogram approach and searching around the previously detected line was implemntned as suggested in the assignement material. For the line sanity check I included:
* checking if line are parallel (I look for abnormal standard deviation distance between pixels that create lane lines) 
* checking avg lane width
* checking lane width at top and bottom

Additionally I experimented with sanity check based on curveture but did not work particularly well.

![alt text][image5]

#### 5. [Curvature] Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

For calculating curvature and car position I followed similar code as shown during the class. No major additions were made.

#### 6. [Final Output] Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

After putting all steps together and constructing the pipeline I experimented with the examples images, still frames extracted from the assignement videos and full length videos. Below can be seen the final output for one of the example images.

![alt text][image5]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_annotated.mp4)

#### 2. Apart of the regular video, I also provide a solution for the challenge video:

Here's a [link to my video result](./challenge_video_annotated.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The **"test_images"** being easiest were annoatated properly with simple pipeline without memory of historical frames and gradient+HLS transforms. For the **project_video** to improve performance and get rid of issues I added RGB transform, searching around the line detected in previous frames, line sanity check and usage of previously detected line in case the check failed. Finally, for the **"challenge_video"** there was a lot of noise or insufficient pixels detected depending on lighting conditions (e.g. under the bridge), also some problems surfaced due to darker lines on the road. To cope with those I investigated more other color transforms and settled for L from HLS, R+G from RGB and no sobel gradient at all.

I didn't continue with the hardest video as this project was already taking too much time with respect to overall course duration. 

Some potential improvements I can see: a) further dig into the color transformation and thresholds; b) remove the line wobbling by averaging several past lines; c) impove the sanity checks.
