## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

## Rubric Points
Link to [rubric points](https://review.udacity.com/#!/rubrics/571/view) 

### 1. Camera Calibration
#####  a. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image

The code for the camera matrix and distortion can be found in the Jupyter notebook project.ipynb (first 2 code cells).

A number of chessboard images are sampled in the camera_cal folder where, images were taken from different angles with the same camera. The goal of this section of th eproject is to undistort the distorted images and find the calibration matrix and distortion coefficients.

First cell in the notebook explains how the obj points, which are 3D points in real world are mapped to the img points, 2D points on the camera image. Mapping is done by finding the corners using the OpenCV functions findChessboardCorners and drawChessboardCorners.

If we notice the images, we see image calibration 1, 4, and 5 are empty. This is because the original image corners are hidden/out of the image scope. We had input 9X6 corners to the program and as program was not able to see enough corners it displayed blank images.

Preview of corner drawn on images

 ![png](./images/findCorners.png)

#### b. Apply a distortion correction to raw images.
Code for correcting distorted images can be found in cell 3, where OpenCV calibrateCamera and undistort function were used. Opencv function, calibrateCamera takes obj and img points as input and return the callibration matrix and distortion coefficients as output. These are stored in calibration.p. 

undistort function undistorts the effects of distortion on any image. It takes distorted image, callibration matrix and distortion coefficients as input and return undistorted image as output.

Preview of undistoretd chessboard image
 ![png](./images/calibrate.png)

### 2. Pipeline (Single test Images) 
 #### 1. Provide an example of a distortion-corrected image.
 As a first step we need to convert the original distorted image to undistorted image. Code for correcting distorted pipeline images can    be found in cell 5. 
 
 Preview of undistoretd test image.
 ![png](./images/undistortion.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

##### a. Examples of Color transforms:
Code to identify which color channels would yield better results in identifying the lane lines can be found in cell 7.

 Preview of various channels of three different color spaces for the same image. It is identified that
* RGB, R channel performed better in identifying lane lines
* HLS, S, L channel seems to recognize both Yellow and white lines better
* HSV, S channel seems to recognize both Yellow and white lines better.

 Preview of color channels:
 ![png](./images/color.png)

 
 #### b. Examples of Thresholds

 I decided to use below thresholds to identify which of these would yield better results in outputting better binary image.
 #### color threshold 
    * R and G channels, so yellow lanes are detected well. (Cell 13)
    
 ![png](./images/rgchannel.png)
 
    * S channel performs well for detecting bright yellow and white lanes. (cell 15)
    
 ![png](./images/schannel.png)

    * L channel to avoid pixels which have shadows. (Cell 15)
    
 ![png](./images/lchannel.png)
  #### Gradients
     * combined sobel(along x direction) and direction gradient, as it was highlighing the lane lines more. (Cell 11)
     
 ![png](./images/combined.png)
   
   After trying all the thresolds which I have learned from Udacity. combination of above color threshold(RG, S, L channels) and Gradients ( sobel + Direction) Thresholded gave better results. 
   
  Preview of the final thresholded image.
   
![png](./images/threshold.png)
   
   #### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
   
![png](./images/perspective.png)

After applying calibration, thresholding, and a perspective transform to a road image, we get the binary image where the lane lines stand out clearly.

The code for my perspective transformed binary image is in cell 23. The openCV function unwarp() takes image,  source (src) and destination (dst) points as inputs. I picked 4 points in the source image which will form a trapezoid that would represent a rectangle when looking down on the road from above. 

  #### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
  
 After applying calibration, thresholding, and a perspective transform to a road test image, the resulted binary imagelane lines stand out clearly. But we still need to decide which pixels are part of the lines and which belong to the left line right line.

 #### Histogram:

As a first step I took a histogram along all the columns in the lower half of the image like this:

![png](./images/histogram.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Functions sliding_window_polyfit and polyfit_using_prev_fit, identify lane lines and fit a second order polynomial to both right and left lane lines.  As a first step in identifying the lanes I computed the hostogram of the bottom half of the image to find the bottom most X position of the left and right lanes. In the Udacity lectures these locations were identified from the local maxima of the left and right halves of the histogram, but in my implementation I identifies using quarters of the histogram just left and right of the midpoint to reject lines from adjacent lanes. The image window was broken down to 10 windows to identify lane pixels, each one centered on the midpoint of the pixels to effectively follow the lane lines up to the top of the binary image. This speeds processing by only searching for activated pixels over a small portion of the image.The image below demonstrates how this process works:

![png](./images/histogram.png)

The polyfit_using_prev_fit function performs basically the same task as sliding_window_polyfit, but alleviates much difficulty of the search process by leveraging a previous fit (from a previous video frame, for example) and only searching for lane pixels within a certain range of that fit. The image below demonstrates this - the green shaded area is the range from the previous fit, and the yellow lines and red and blue pixels are from the current image:
![png](./images/histogram.png)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is based upon this website and calculated in the code cell titled "Radius of Curvature and Distance from Lane Center Calculation" using this line of code (altered for clarity):

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])

In this example, fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
I implemented this step in the code cells titled "Draw the Detected Lane Back onto the Original Image" and "Draw Curvature Radius and Distance from Center Data onto the Original Image" in the Jupyter notebook. A polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix Minv and overlaid onto the original image. The image below is an example of the results of the draw_lane function:
![png](./images/histogram.png)

Below is an example of the results of the draw_data function, which writes text identifying the curvature radius and vehicle position data onto the original image:
![png](./images/histogram.png)

### Pipeline (video)

1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
