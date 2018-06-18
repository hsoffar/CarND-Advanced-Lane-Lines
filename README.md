## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[BIRDEYE]: ./pictures/birdeye.jpg "BIRDEYE"
[calib]: ./pictures/calib.jpg "calib.jpg"
[colortres]: ./pictures/colortres.jpg "colortres"
[final]: ./pictures/final.jpg "final"
[sliding]: ./pictures/sliding.jpg "sliding"
[undist]: ./pictures/undist.jpg "undist"

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

The images for camera calibration are stored in the folder called `camera_cal`.  The images in `test_images` are for testing your pipeline on single frames.  If you want to extract more test images from the videos, you can simply use an image writing method like `cv2.imwrite()`, i.e., you can read the video in frame by frame as usual, and for frames you want to save for later you can write to an image file.  

To help the reviewer examine your work, please save examples of the output from each stage of your pipeline in the folder called `output_images`, and include a description in your writeup for the project of what each image shows.    The video called `project_video.mp4` is the video your pipeline should work well on.  

## Camera calibration
every camera comes with a little bit of distortion in the images it puts out. Since each camera's distortion can be different, a calibration must be done in order to correct the image and make it appear as in the real-world, undistorted. Luckily, OpenCV provides an easy way to do so using chessboard images taken by the camera. I pulled in some of Udacity's provided chessboard images from their camera to begin with.
![alt text][calib]
Using cv2.findChessboardCorners, the corners points are stored in an array imgpoints for each calibration image where the chessboard could be found. The object points will always be the same as the known coordinates of the chessboard with zero as 'z' coordinate because the chessboard is flat. The object points are stored in an array called objpoints. I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera function.

We start by preparing "object points", which are (x, y, z) coordinates of the chessboard corners in the world (assuming coordinates such that z=0). Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

objpoints and imgpoints are then used to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:

## distortion 
Applying the undistortion transformation to a test image yields the following result (left distorted, right corrected) 
![alt text][undist]
## Binary lane line image using gradient and color transforms

![alt text][colortres]
## Perspective Transform to bird's eye view
![alt text][BIRDEYE]
## Identifying lane line pixels using sliding windows
![alt text][sliding]

## Image pipe Line final

![alt text][final]
## Video Processing Pipeline


## Discussion
# Problems encountered and Outlook
Getting the pipeline to be robust against shadows and at the same time capable of detecting yellow lane lines on white ground was the greates difficulty. I took the approach that lines should never have very low saturation values, i.e. be black. Setting a minimal value for the saturation helped when paired with the x gradient and absolute gradient threshold. Problematic is also the case when more than two lanes get detected. For lines detected far away a threshold on the distance eliminated the problem. By far the most work was implementing the logic of a continuous update of detected lines as well as restarting when the buffer of previous lines emptied. At the moment the pipeline will likely fail as soon as more (spurious) lines are on the same lane, as e.g. in the first challenge video. This could be solved by building separate lane line detectors for yellow and white together with additional logic which line to choose.


