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

The code for the camera matrix and distortion can be found in the Jupyter notebook project3.ipynb (first 2 code cells).

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

The code for my perspective transform is titled "Perspective Transform" in cell 23. The openCV function unwarp() takes image,  source (src) and destination (dst) points as inputs. I chose to hardcode the source and destination points in the following manner

