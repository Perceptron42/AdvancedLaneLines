
# Advanced Lane Finding Project


![Alt Text](https://media.giphy.com/media/wbqFb7H8o3iGja115e/giphy.gif)

Here's a [link to my video result on youtube.](https://www.youtube.com/watch?v=4fu25dC8uTY)


## Pipeline Description

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

[image1]: ./camera_cal/test_undist.jpg "Undistorted"
[image2]: ./test_images/test2.jpg "Road Transformed"
[image3]: ./output_images/threshold_images/test2.jpg "Binary Example"
[image4]: ./output_images/birds_eye_view_images/test2.jpg "Bird's eye view example"
[image5]:./output_images/draw_lane_lines/test2.jpg "DrawLaneLines"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for camera calibration is contained in separate ipython notebook since once the camera is calibrated we do not have to recalibrate it.
 
The name of the notebook is "Calibrate Camera with OpenCV.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  


I started with object points and images points. 

The chess board corners are found using inbuild OpenCV function called "findChessboardCorners".

Once the object points and images points are found we can use the following OpenCV function to calculate the camera matrix and distortion coefficients. 

``` python
ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, img_size,None,None)
```
The following example image is produced using ```undistort``` function from openCV which takes in image, camera matrix and distortion coefficients. 


![Original and Undistored chessboard image][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I read in the camera matric and distortion coefficients from a pickle file and used the ```undistort``` function to undistort a single image. This is in done in cell 4. Here is the output of an undistorted image. 

![Undistorted Image][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I first converted the image into HLS color space using ```cvtColor``` function and then  used a combination of X gradient and S channel thresholding to get a thresholded binary image. This is done in cell 6 of the AdvanedLaneFinding iphython notebook (see function ``` threshold_image ``` ). 

Here is an example of the binary image result.
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I assumed the road is a flat place. Similar to the region masking I picked four points which are in trapezoidal shape. This step was very tricky and had to play with many source points. 

Destination points are straight forward since they will be in the shape of a rectangle. 

Source points are referring to the original undistorted image and destination points are corresponding points in the bird's eye view image. 

Source and destination points are fed into the ```getPerspectiveTransform``` function first to ge the M matrix. Then This M matric is applied to the orignal image using ```warpPerspective``` function. 

 This is done in cell 8 of the AdvanedLaneFinding iphython notebook (see function ``` perspective_transform ``` ). 

Here is an example of the bird's eye view image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For a single image I used lower half of the image to figure out the lane lines starting point on the bottom of the image. i disregarded the 50 pixels since there could be a divider on the left side of the image. I am assuming the car will always be driving on the right side of the road. Then with the x coordinates of the starting point of the lane lines I applied the sliding window algorithm. I chose number of windows to be 9 and the width of the windows to be 100 pixels wide.  the Current window gets recentered if we find greater than 50 lane pixels. 

I searched for active pixels in the 9 windows. Pixels were stores in left and right lane array of indices (```left_lane_inds```, etc.) and then passed into numpy `polyfit` function to get a second order polynomial. 

This is done in cell 10 of the AdvanedLaneFinding iphython notebook (see function ``` fit_polynomial_first_lane``` which calls ``` first_lane_pixels``` ). 

Here is an example image of active left lane (red) and right lane (blue) pixels. The fitted 
polynomial is in yellow. 

![alt text][image5]

Note that for the video I used another function called ```search_around_poly``` which uses polynomial of the fitted lines in the previous frame and uses that polynomial as a guide to search for new pixels. This results in easier and faster search of lane line pixels. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This is done in cell 13 of the AdvanedLaneFinding iphython notebook (see function ``` measure_curvature_real``` ). 

I calculated the curvature using the following steps:

 - calculate new coordinates in world space from pixel space using a scaling factor and the fitted lines. 
 - fit the new coordinates using polyfit
 - calculate the radius of curvature using the new coefficients of the fitted polynomial in the real world space
 

 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is done in cell 14	 of the AdvanedLaneFinding iphython notebook (see function ``` draw_on_original_image``` ). 

Following are the main steps:

 - on a blank image draw the fitted points and use fillPoly function to
   color the area. 
 - Unwarp the image from bird's eye view to camera view
   using Minv (inverse Matrix) calculated in earlier steps.
 - Combine the unwarp and
   original image.

This is am example of the results plotted back:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result on youtube.](https://www.youtube.com/watch?v=4fu25dC8uTY)
![Alt Text](https://media.giphy.com/media/wbqFb7H8o3iGja115e/giphy.gif)

Following individual videos can be found in the root directory: 
-- project_output_v01_combined_binary.mp4 (thresholding step video)
-- project_output_v01_perp_transform.mp4 (bird's eye view video)
-- project_output_lane_lines.mp4 (lane lines searched)
-- project_output_v01.mp4 (Final Video)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

## Project Shortcomings
Following are the shortcomings of this project:
1. The code doesnt perform well in a lot of brightness and or changes to brightness such as shows in the challenge video.
3.  In the perspective transform function we are assuming road is a flat place which is strictly not true. So this code will have problems when the road is sloping up or down.
4. I disregarded the 50 pixels since there could be a divider on the left side of the image. I am assuming the car will always be driving on the right side of the road. This will give issue if we are using a video where either the road is curving steeply to the left or we are in a country where the vehicles are driving on the left side of the road. 

## Possible Improvements 

Following possible improvements can be made to the code:
1. In the first lane line detection we can use a seperate color space to find independent lane lines and acerage the results of that color space to the current one in the code. This would make lane lines more robust. 
2. In the perspective transfom (bird'd eye view) step I am using the hardcoded source and destination points. It won't work well if they image size is different that the current video. So we can use source and destination points pased on the shape of the image passed. 
3. In the video to stablize lane lines I am taking an average of last 15  frames to stablize the lines. I think it is better to take a longer average of like 30-40 frames but give more weight to more recent lines. We can also reject lines if they are deviating a lot from this average. 
