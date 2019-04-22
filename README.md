## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[image1]: ./examples/undistort_output.png "Undistorted"
[testImage]: ./output_images/test1Img.png "testImage" 
[testUndistImg]: ./output_images/testundistimg.png "testUndistImg"
[testwarpImg]: ./output_images/testwarpimg.png "testwarpImg"
[testwarpRect]: ./output_images/testwarp_rectImg.png "testwarpRect"
[testLImg]: ./output_images/testLimg.png "testLImg"
[testSImg]: ./output_images/testSImg.png "testSImg"
[testBImg]: ./output_images/testBimg.png "testBImg"
[testLSBImg]: ./output_images/lsbcombined.png "testLSBImg"
[colorThresh]: ./output_images/ColorThresholdImages.png "colorThresh"
[sliding]: ./output_images/slidingwindow.png "sliding"
[checkerBoard]: ./output_images/checkerboard.png "checkerBoard"
[perspective]: ./output_images/perspectiveImgs.png "perspective"
 

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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
1. Environment Setup
Usually when developing code for any project, I setup the environemnt. Refer to AdvLaneFindingProjectComplete1.ipynb for Import Packages and DEFINES sections. 

2. Camera Calibration: 
Udacity provided bunch of images of checkboard pattern and we were required to use cv2.CalibrateCamera. All the images were read and processed in the function getObjnImgPoints which in turn uses cv2.findChessboardCorners to return important points for the checkerboard images. Once the camera is calibrated the result is used to undistort the sample images (or even actual images), using camera calibration coefficients. Result of undistorted image are shown below.  

![alt text][checkerBoard] 

3. Pipeline: 
In very broad terms the pipeline is essentially as shown below. In the project code, I have worked on single images to qualify my code and robustly identify the lanes. As these functions were developed, they were then effectively used for the final video file as well. 

#### a. Apply a distortion correction to raw images
The first step for the image is to undistort the image. This was done based on the camera calibration coeffients identified in the earlier section. We assumed that the same camera was used to capture the images 

![alt text][testImage] ![alt text][testUndistImg]

#### b. Perspective transform
Once the undistorted image was obtained, then I created a birdseye view for the same image. This will allow use to look at the road thus making lane identificatoin easier. The trick here is finding the correct points to use for the transform. I used visual aid to decide which points from the undistorted image will be mapped to create the perspective transform. Example shown below

![alt_text][testwarpRect]

These points are as follows: 

| Source        | Destination   | 
|:-------------:|:-------------:| 
|  575, 460     | 450, 0        | 
|  750, 460     | 830, 0        |
|  250, 700     | 450, 720      |
| 1100, 700     | 830, 720      |

The birdeye view looks like this: 

![alt_text][testwarpImg]

Here's a look at all the test image's perspective view: 
![alt_text][perspective]

#### c. Create a thresholded binary image
I spent a lot of time in this particular section. The main reason was that I faced a lot of challenged when applying sobel transform. while the magnitude transform gave me okay results for most of the images, test4, test5 failed miserably. I also had doubt on whether I should be applying sobel on the perspective transform image or the undistorted image. All in all, using sobel transform caused my lane detection to go completly wrong. Thus, I decided to seek alternative approaches. I came across various blogs and implementation on using color transform alone to detect the lanes. While it may not be the objective of this assginment, I decided to try out HSL, HSV and LAB transform. Finally after trying a lot of combinations, I realized that  L, S and B images gave me most satisfactory result. Here is a sample of images I looked at to select my color threshold. 

![alt_text][colorThresh] 


My final selection was:
L threshold from HSL transform 

![alt_text][testLImg]

S threshold from HSL transform

![alt_text][testSImg]

B threshold from LAB transform

![alt_text][testBImg]

The combined threshold image for test image is shown below: 

![alt_text][testLSBImg]



#### Lane detection

#### Determine curvature and vehicle offset

#### Overlay lane on orignal image

#### Display information on orignal image


The images for camera calibration are stored in the folder called `camera_cal`.  The images in `test_images` are for testing your pipeline on single frames.  If you want to extract more test images from the videos, you can simply use an image writing method like `cv2.imwrite()`, i.e., you can read the video in frame by frame as usual, and for frames you want to save for later you can write to an image file.  

To help the reviewer examine your work, please save examples of the output from each stage of your pipeline in the folder called `output_images`, and include a description in your writeup for the project of what each image shows.    The video called `project_video.mp4` is the video your pipeline should work well on.  

The `challenge_video.mp4` video is an extra (and optional) challenge for you if you want to test your pipeline under somewhat trickier conditions.  The `harder_challenge.mp4` video is another optional challenge and is brutal!

If you're feeling ambitious (again, totally optional though), don't stop there!  We encourage you to go out and take video of your own, calibrate your camera and show us how you would implement this project from scratch!

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

"# CarNDAdvancedLaneFinding" 
