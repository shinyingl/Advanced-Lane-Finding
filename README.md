# Self Drivng Car - Project 2:  Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](output_images/08_Final2.png)

The goal of the project is to write a software pipeline to identify the lane boundaries in a video. 


The Project Outline
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


### Link of Python Code: 
https://github.com/shinyingl/SelfDrivingCar-P2/blob/master/Project2-Advanced-Lane-Lines.ipynb

### 1. Camera Calibration
---
The camera calibration code is in `def CameraCal(images)`. Images of chessboard pattern captured at different angles is used to calibrate the abberations from the camera optical system. The undistorted pattern after calibration is shown as the below images.
![Image1](output_images/01_Undistrot.png)
![Image2](output_images/02_Undistrot.png)

### 2. Thresholded Binary Image
---
- Color Transform (`def color_thresh`): The images are converted into HLS color space. L channel and S channel are both used. I found limiting L channel helps to stablize the lane line color binary output under tree shadows. 
- Grandient Transform (`def abs_sobel_thresh` & `def mag_thresh` & `def dir_threshold`): Consider Sobel gradient in horizonal/vertial, magnitude, line orientation. 
- A combinarion of the above transform were used for the binary image output is done in `def combin_thre`.
![Image3](output_images/03_CombinedThreshold.png)

### 3. Perspective Transform (*birds-eye view*)
---
The image "straight_line1.jpg" is used to obtain the tranformation matrix M and inverse of M in `def birdseye`. It is easier to eye-ball if the transformation is working as expecected. The lane lines from the image are transform into a birds-eye view that shows the two lane lines are in parrellel. I assume this part can potentially be done by physically measuring the camera tilting angle. 
*After the 1st submission of the project, I was reminded the lane is shifted after the perspective transform. I also noticed that the scale was changed after the tranformation. This shows impact on the curvature radius calculation car center offset later in the pipleline. The dst is updated to match the physical dimension conversion*
![Image4](output_images/04_BirdsEyeView.png)

### 4. Find Lane Line at T0
---
Search the first frame lane lin with `def find_lane_line`:
- Histogram: The initial search of the lane line is done with the histogram of bottom 1/3 of thresholded binary images. 
- Sliding Window: A reasonable size of window starts from the bottom of the image and slides all the way to the top of the image by recentering the window based on the non-zero pixels found in the window. The window size should be determined by the lane line curvature of the image.
- Polynomial Fit: Use a 2nd order polynomial to fit the non-zero piexels in the sliding windows. Draw the lane line boudary from the fitted curve in yellow lines. 

![Image5](output_images/05_FindLaneBoundary.png)   ![Image5](output_images/05_Histogram.png)

### 5. Find Lane Line from the Feedback of Previous Frame
Use previous frame information as initial conditions to search current frame lane line with `search_around_poly`:
- From the 2nd frame of the video, use the coefficient from previous frame as starting point to search non-zero pixels in the green shaded area as below. Same searching ranged is used as in `def find_lane_line`.
- Do polynominal fit to predict the lane line boundary. 
- Use the feedback from previous frame increases the stability of the lane line prediction.
![Image6](output_images/06_FindLaneBoundary.png)

### 6. Determine the Curvature and Car Offset
Curvature radius is calculated in `def measure_curvature_real`, the car location with respect to image center is calculated in  `def car_offset`
- The pixel number coordinate is converted into physical unint in meters.
- The curvatures is calculated with the eaqution: <img src="output_images/11_Curvature.png" width="20%">
- The curvatures of both lanes are calculated from the location at the bottom of the image. The output curvature is calculated by averaging the curvalture of both left lane line and right lane line.
- The car offset location is calculated by *the base* of left lane line and right lane line x-location subtracted by the image mid-point in x-direction.

  ![Image7](output_images/07_NumericalOutput2.png)

### 7. Final Pipeline & Output in Static Image
By putting all the above together, a final image processing pipline is as show below:
![Image8](output_images/10_Pipeline.png)

Test the pipeline on some static frames (that I had trouble with in the first few versions of pipeline):

![Image9](output_images/09_Issue2_2.png) ![Image10](output_images/09_Issue1_2.png) 

### 8. Project Output Video
- Project Video Link: 
	- (2nd submission) https://github.com/shinyingl/SelfDrivingCar-P2/blob/master/project_video_out_final2.mp4 
    - (1st submission) https://github.com/shinyingl/SelfDrivingCar-P2/blob/master/project_video_out_final.mp4

### 9. Challenge
The same pipeline is applied on the harder challenge videio. However, it is not very successful. So the video is not uploaded. 
- Below is the first frame result, it indicates further tuning on the threshold and sliding window search is needed. 
- My green-shade marked lane completely disappered after some light glare comes into the videio. It might be able to be improved if I input more previous frame data into the image processing pipeline if the current frame lane line is not found. 
- I do just notice here are some more suggesions at the end of project rubric page. But I will stop here for now and add the link as my own note for future improvment. Link: https://review.udacity.com/#!/rubrics/1966/view

![Image11](output_images/12_Challenge.png) 

