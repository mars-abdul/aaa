## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./examples/example_output.jpg)

In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  


The Project
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I followed the following steps for distortion correction:
* First read all the caliberations images and found chessboard corners in (9,6) matrix configuration.
* Using cv2.calibrateCamera() function got camera matrix, distortion coefficients, Rotational vectors and translational vectors.
* Used the above parameters to undistord the image. These parameters were pickled to directly use on lane images

* Below is the exaple of distorted image converted to undistorted image

<p>
 <img src="./output_steps/original_chess_cal.png" width="40%" height="40%">Original
 <img src="./output_steps/distortion_corrected_cal.png" width="40%" height="40%">Undistorted
<br>
</p>

### Pipeline (test images)

<p>
 <img src="./output_steps/sample_lane.png" width="40%" height="40%">Original
<p>
  
#### 1. Provide an example of a distortion-corrected image.
Used the pickled mtx and dist parameters to undistort lane images.

<p>
 <img src="./output_steps/undistort_lane.png" width="40%" height="40%">undistort_lane
<p>
  
#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

* I  applied hls transform in apply_hls() function using cv2.cvtColor(img, cv2.COLOR_RGB2HLS) function. Then I selected S- channel and applied a thresholf of (170, 255).
* I also applied gradient transform using sobel function in the x-axis. And applied a threshold of (15, 100).
* Then  combined the above 2 color and gradient transforms to  onbtain the transformed image with lane lines.

<p>
 <img src="./output_steps/color_grad_th_lane.png" width="40%" height="40%">color_grad_lane
<p>
  
#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Performed the perspective transform in perspective_transform() function using cv2.getPerspectiveTransform(src, dst).
Here src and dst are the four points identified on binary src image to be transformed to dst bird-eye view image.
This  helps in identifying curved lanes very eaisly.

<p>
 <img src="./output_steps/bird_view_lane.png" width="40%" height="40%">bird_view_lane
<p>

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Lane lines are identified in findLines() function. 

* I use histogram to ascertain peaks along the x-axis of the perspective transformed images. First image is divided into equal halves and the histogram peaks are identified as left and right lane lines starting points.
* Then i used sliding windows to slide up the lane lines and identify points for lane lines. 
* A value of minimum pixels (set as 50) determines the sliding of window. When minimum number of lane pixels are identified in new window, the left and right lane values are appended.
* Thus we obtain list of points for left and right lanes. 
* These are fit to a second order polynomial using np.polyfit function and can be plotted on the binary as shown below.

<p>
 <img src="./output_steps/fit_lines.png" width="40%" height="40%">fit_lines
<p>
  
#### 4. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To fit the image to real world distances, I took 30/720 meters per pixel in x- direction and 3.7/700 meters per pixel in y- direction.

* Using the above convertion factor, I fit the lane lines in real world.
* Using the polynomial coefficients, radius of curvature is calculated using formulae in calculate_values() method. Similarly position of vehicle is also identified.

#### 5. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The below final image shows the identified Lane Area.

<p>
 <img src="./output_steps/marked_lane.png" width="40%" height="40%">marked_lane
<p>
  
### Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)

The video is stored in output_videos folder.

### Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

* Problems faced - 
1. I faced some issues in sliding windows on how it actually works. And this took some time to understand.
2. Identifying proper thresholds for binary took some time as the lane lines were not getting properly detected. Solved the issue by visualizing the results and experimenting with different values.

* Pipeline challenges - 
1. Detected lane is slightly tapering at the top on turning right lane.
2. In extreme lighting condition, detected lanes have some degree of noise.

I think these can be minimised if I average the lane lines over historical n-frames.
