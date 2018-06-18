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

## Distortion 
Applying the undistortion transformation to a test image yields the following result (left distorted, right corrected) 
![alt text][undist]
## Binary lane line image using gradient and color transforms
I explored several combinations of sobel gradient thresholds and color channel thresholds in multiple color spaces. These are shown clearly in the project Jupyter notebook(pipeline). Below is an example of the combination of sobel magnitude and direction thresholds and HSL tresholding, the porpuse of this step was to obrain clear driving lane lines.
![alt text][colortres]
## Perspective Transform to bird's eye view
The Birds_eye() function takes as inputs an image (img), as well as source (src) and destination (dst) points. I chose to hardcode the source and destination points in the following manner:
  src = np.float32([[490, 482],[810, 482],
                      [1250, 720],[40, 720]])
    dst = np.float32([[0, 0], [1280, 0], 
                     [1250, 720],[40, 720]])
The goal of this step is to transform the undistorted image to a "birds eye view" of the road which focuses only on the lane lines and displays them in such a way that they appear to be relatively parallel to eachother (as opposed to the converging lines you would normally see). To achieve the perspective transformation I first applied the OpenCV functions getPerspectiveTransform and warpPerspective which take a matrix of four source points on the undistorted image and remaps them to four destination points on the warped image. The source and destination points were selected manually by visualizing the locations of the lane lines on a series of test images.
![alt text][BIRDEYE]
## Identifying lane line pixels using sliding windows
After applying calibration, thresholding, and a perspective transform to a road image, you should have a binary image where the lane lines stand out clearly. However, you still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.
I first take a histogram along all the columns in the lower half of the image , to clearly identify the lane pixels or the point where we should start searching with the sliding window method.
Sliding Window
With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.
![alt text][sliding]

## Image pipe Line final
Here is the ouput of the pipe line used on the sample image.
Apply undistorion to the image
    image = cal_undistort(img_c, objpoints, imgpoints)
apply sobel threshold 
    gradx = abs_sobel_thresh(image, orient='x', sobel_kernel=3, thresh=(15, 255))
    grady = abs_sobel_thresh(image, orient='y', sobel_kernel=3, thresh=(35, 255))
Apply Magnitude and direction threshold
    mag_binary = mag_thresh(image, sobel_kernel=9, mag_thresh=(60, 255))
    dir_binary = dir_threshold(image, sobel_kernel=9, thresh=(0.7, 1.1))
Apply Color tresholding using HSL color space
    color = color_thresholds(image)
Combine threshold images and get a binary images , which should contain the lane lines
    combined = np.zeros_like(img_g)
    combined[(gradx == 1) & (grady == 1) & (mag_binary == 1) | (color == 1) | (mag_binary == 1) & (dir_binary == 1)] = 1
Perform birdEye view transformation
    birds , Minv = birds_eye(combined, display=False)
Apply the Sliding window search to get the Lanes
    windows_img, ploty, left_fitx, right_fitx, left_fit, right_fit, leftx, rightx, leftx_base, rightx_base = sliding_windows(birds)
Obtain curveature of the road
    left_curverad = roc_in_meters(ploty, left_fit, right_fit, leftx, rightx)
Get the camera position
    center_pos =  pos_center(img.shape[1]/2,leftx_base,rightx_base)
    annotate(img, left_curverad,  center_pos)
Draw lines
    result = draw_lane(img ,birds,left_fit, right_fit, Minv)

Obtain the following results
![alt text][final]
## Video Processing Pipeline


## Discussion
# Problems encountered and Outlook
Getting the pipeline to be robust against shadows and at the same time capable of detecting yellow lane lines on white ground was the greates difficulty. I took the approach that lines should never have very low saturation values, i.e. be black. Setting a minimal value for the saturation helped when paired with the x gradient and absolute gradient threshold. Problematic is also the case when more than two lanes get detected. For lines detected far away a threshold on the distance eliminated the problem. By far the most work was implementing the logic of a continuous update of detected lines as well as restarting when the buffer of previous lines emptied. At the moment the pipeline will likely fail as soon as more (spurious) lines are on the same lane, as e.g. in the first challenge video. This could be solved by building separate lane line detectors for yellow and white together with additional logic which line to choose.


