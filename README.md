## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[image1]: ./examples/undistort_output.png "Undistorted"
[testImage]: ./output_images/testimage.png "testImage" 
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
[testColorThresh]: ./output_images/colorThresholdImages_test.png "testColorThresh"  
[hist]: ./output_images/testhist.png "hist"
[polyfit]: ./output_images/testployplot.png "polyfit"
[printdata]: ./output_images/testLanemark_data.png "printdata"
[lanemarking]: ./output_images/lanemarking.png "lanemarking"


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
### 1. Environment Setup
Usually when developing code for any project, I setup the environemnt. Refer to AdvLaneFindingProjectComplete1.ipynb for Import Packages and DEFINES sections. 

### 2. Camera Calibration: 
Udacity provided bunch of images of checkboard pattern and we were required to use cv2.CalibrateCamera. All the images were read and processed in the function getObjnImgPoints which in turn uses cv2.findChessboardCorners to return important points for the checkerboard images. Once the camera is calibrated the result is used to undistort the sample images (or even actual images), using camera calibration coefficients. Result of undistorted image are shown below.  

![alt text][checkerBoard] 

### 3. Pipeline [Test Images]: 
In very broad terms the pipeline is essentially as shown below. In the project code, I have worked on single images to qualify my code and robustly identify the lanes. As these functions were developed, they were then effectively used for the final video file as well. 

#### **a. Apply a distortion correction to raw images**
The first step for the image is to undistort the image. This was done based on the camera calibration coeffients identified in the earlier section. We assumed that the same camera was used to capture the images 

![alt text][testImage] ![alt text][testUndistImg]

#### **b. Perspective transform**
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

#### **c. Create a thresholded binary image**
I spent a lot of time in this particular section. The main reason was that I faced a lot of challenged when applying sobel transform. while the magnitude transform gave me okay results for most of the images, test4, test5 failed miserably. I also had doubt on whether I should be applying sobel on the perspective transform image or the undistorted image. All in all, using sobel transform caused my lane detection to go completly wrong. Thus, I decided to seek alternative approaches. I came across various blogs and implementation on using color transform alone to detect the lanes. While it may not be the objective of this assginment, I decided to try out HSL, HSV and LAB transform. Finally after trying a lot of combinations, I realized that  L, S and B images gave me most satisfactory result. Here is a sample of images I looked at to select my color threshold. 

![alt_text][colorThresh] 

My final selection was:
L channel from HSL transform 

![alt_text][testLImg]

S channel from HSL transform

![alt_text][testSImg]

B channel from LAB transform

![alt_text][testBImg]

The combined threshold image for test image is shown below: 

![alt_text][testLSBImg]


The output for all the test images is shown below: 

![alt_text][testColorThresh]

#### **d. Lane detection** 
During the course there were two mechanisms taught for detecting the lanes. The idea was identifying the points of the lane within a certain window. After such points are identified, then create a polynomial that fits those points. Once such a polynomial is identified, that can be used for next frames to plot the lanes. However, there are various aspectst to be kept in mind. Robust image filtering being one of them. 
While my approach is reasonable, I do not believe that it is robust enough. This was made quite evident when I run the same algorithm on the challenge and harder challenge videos.

The identification of the lanes was first done by a technique called as sliding windows. The idea is that once lanes starting points have been identified, then the image is divided into multiple smaller areas (windows) and lanes points are searched within those windows. These window slide around based on where the lanes points appear. 
The functions used for this operation are: sliding_window_polyfit 
Here's what it looks like: 

![alt_text][sliding]

How does one know where to look for the lane? Well, since the camera is mounted in the center of the vehicle, in the perspective image, if we take the histogram of the image, peaks are observed where the lanes appear. This gives use the starting point of the lanes. Here's what the histogram for test image looks like: 

![alt_text][hist]

Once the lanes and the polynomials are established, the next frames do not need to use the sliding window approach. We can use the polynomial fit to identify points along the curve within a certain margin and modify the polynomial based on new findings. This is shown in the image below: 

![alt_text][polyfit]


#### **e. Overlay lane on orignal image**

Once the lanes are identified, we need to overlay the lane data on the original image. Thus, we need to warp the image back to real world view. The function that does this is: plotLinesOnFrame
The output image looks like this: 

![alt_text][lanemarking]

#### **f. Determine curvature and vehicle offset** 

The function that is used to calculate the radius of the lane is: measure_curvature_real
Internally it used the fitted polynomial to compute the radius. The reference for this calculation is explained [here](https://www.intmath.com/applications-differentiation/8-radius-curvature.php). 

The formula below calcuates the radius at pixel level, but since we want to know the real world radius, we need to multiply the output with ym_per_pix transform. This is value is (as documented in the code), meters per pixes in y dimension. 

curveRadius = ((1 + (2*left_fit[0]*y_eval*ym_per_pix + left_fit[1])^2)^1.5)/np.absolute(2*left_fit[0]) 

Similarly, the vehicle offset from the center is calculated based on certain assumptions. The main assumption is that the camera is mounted exactly at the center of the vehicle. Second, the image center is basically where the center of the lane is located. 

The function used to compute this information is: CalcVehLaneOffset
The polynomial is used to compute the X and Y intercepts. These are the points located on the bottom of the image. Thus, we know where the left and right lanes intersect the bottom of the image (or the location of the vehicle). Now, half of these two points is where the center of the lane is. Measured with respect to that, the half of the image will tell us the offset of the vehicle from the lane center. 


#### **g. Display information on orignal image**

To display this information on the image, the function used is: printVehRoadData

The output of this function looks like this: 

![alt_text][printdata]

### 4. Pipeline [Video]: 

Video is nothing but just of series of images stringed together. So, essentially we are processing an image of each frame. So, the pipeline that was described in previous section (section 3) is what is leveraged for processing the video. The pipeline function that processes the video is: ProcessVideoFrame. 

This funciton processes each frame to detect the lanes and then will plot the lanes on orignal image and print the lane data on each frame. 
However, there is a small trick that needs to be played. While it is easy to process each image and see the output on the image, when processing a video, how do we know if the data is being processed correctly? We add some robustness techniques to the video processing. If you look at the function "ProcessVideoFrame", you will realize that we are using a class to store the lane information that will be used for the next frame processing. However, we keep a track of multiple such lanes, because the road conditions and lane curvature can change. So, by using the average of previous frames, we are able to add robustness and reduce the frame over frame changes. Otherwise, the polynomial identified in frame 1, will most likely not fit the data by frame number 10 or something similar. This operation can be followed in the function: UpdateLineInfo

The project output video can be found [here](https://github.com/Yaajaam/CarNDAdvancedLaneFinding/blob/master/project_video_output.mp4). 


### 5. Discussion: 

I faced many issues while doing this project. As discussed in the ealier section, the biggest issue I faced was robustness. This issue is evident if my algorithm is used for the challenge and harder challenge videos. I research a lot as to why my code is failing on those videos. There are couple blogs those talk about adding robustness to lane detection. 

[Here](http://petermoran.org/robust-lane-tracking/) is an example of one such blog that I ran into. The author talks about assigning scores to the detected lanes. 
Furthermore, I believe that applying sobel on original image and then transforming that to perspective view is a better approach. This is because that's really where sobel algorithm shines. On a perspective image, the sobel fails miserably, at least that's what I have learned. 
