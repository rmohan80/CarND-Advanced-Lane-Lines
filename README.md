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

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "advancedLaneLines.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

(Note that the results are very subtle and hence don't look all that different)

![Camera Calibration Output](/output_images/camera_calibration.jpeg?raw=true)

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
(Note that again the difference is subtle, but look at the the white car on the right to notice the difference in position)

![Distortion corrected image](/output_images/test_image_undistortion.jpeg?raw=true)

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at code cell 3).  I used the default HLS threshold as provided in the code, but changed the threshold values a little bit to accomodate for the shadows. This ensured that all the images in the examples folder successfully passed this test. Here's an example of my output for this step. 

![Thresholded Binary Image](/output_images/binary_threshold_image.jpeg?raw=true)

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `p_transform_lines()`, which appears in code cell 4. The `p_transform_lines()` function takes as inputs an image (`img`). The source and destination points are hard coded as follows:

Source:
			[240,719], -- bottom left
                        [579,450], -- top left
                        [712,450], -- top right
                        [1165,719] -- bottom right

Destination:
			[300,719],  -- bottom left
			[300,0], -- top left
			[900,0], -- top right
			[900,719]] -- bottom right
	
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Perspective transformed image](/output_images/perspective_transform.jpeg?raw=true)

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used a histogram function to determine the lane line positions and fixed them in the birds-eye view image. This is done via the function `fit_lane()` in code cell 4

![Lane Lines Detection](/output_images/lane_lines.jpeg?raw=true)

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I identified the curvature of the lane via the function `curvature() ` in code cell 6. Calculation of distance from center is part of `fit_lane()` function. See dist_from_center variable 

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is part of code cell 8 function `final_image()`  Here is an example of my result on a test image. As can be seen, it looks great and works on all example images

![enter image description here](/output_images/final_image.jpeg?raw=true)

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](/project_video_output.mp4?raw=true)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The project was fairly straightforward by using the example code provided. All the various bits were there, but I had a bit of challenge in putting it all together because without the sliding window implementation, the ETA for running on every frame was hours for just a minute of video.

I used the default HLS binary threshold method with just some experimentation on the thresholding and it gave me great results. The fact that I'm averaging over several frames and dropping off frames makes this pipeline quite good. However, it fails a little in the challenge video due to 2 major reasons (Here's a [link to my video result](/challenge_video_output.mp4?raw=true)) :

1 - The hardcoding of the source and destination should be replaced by more generic methods using size of the image (couldn't test due to lack of time)

2 - The pipeline has major issues with recognizing the shoulder vs lane marking on the left

The algorithm also has a bit of issues with shadows and differing light conditions. This is swept under the carpet in the default video, but rears its ugly head in the harder challenge video (Here's a [link to my video result](/harder_challenge_video_output.mp4?raw=true)) where it fails miserably.

There is also some issue with putting the lines back on the warped image with shadows coming on the sides of the road (its unclear whether this is my local laptop issue or not)

Maybe tinkering with the frame averaging as well as the threshold will produce better results. Also, it takes quite a bit of time to process the video, maybe the algorithm should be made more efficient so that it can be used in realtime conditions

