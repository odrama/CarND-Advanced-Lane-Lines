## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

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

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.png "Fit Visual"
[image6]: ./examples/example_output.png "Output"
[image7]: ./output_images/output_2_1.png "Calibration"
[image8]: ./output_images/output_6_1.png
[image9]: ./output_images/output_6_2.png "Road thresholded"
[image10]: ./output_images/output_6_3.png "Road thresholded & transformed"
[image11]: ./output_images/output_7_0.png "Test images pipelined"
[image12]: ./output_images/output_9_1.png 
[image13]: ./output_images/output_9_2.png
[image14]: ./output_images/output_11_1.png "Sliding window"
[image15]: ./output_images/output_13_1.png "Sliding window based on prev"
[image16]: ./output_images/output_17_1.png
[image17]: ./output_images/output_17_2.png "Lane super-imposed"
[image18]: ./output_images/output_19_1.png "Lane super-imposed with info"
[image19]: ./output_images/output_3_1.png "Undistorted"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  



### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image7]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

Undistorted would like like: 
![alt text][image19]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I thresholded the Red and Green channels, and bitwise-anded them to detect yellow lanes. I used the L channel from the hls version of the image as it works well even when there are different lighting conditions. And used the s channed along with the direction and x gradient of the image to detect the lanes. I then bitwise-ored them to obtain a robust thresholding function that works well throughout the video. A trapezoidal mask was used to remove outliers and only show the lane section of the image, which contains all the information needed. Function name : `get_thresholded()`

![alt text][image9]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I use a function that undistorts, thresholds, and then warps the images provided as input. Simple opecv getPerspective and warpPerspective functions were used, by provided the camera calibration matrix and correction coefficients obtained in the camera calibration stage.

Trapezoidal polygon source points were used, and rectangular destination points were used. Function name: `undistort_warp()`


![alt text][image10]

All test images thresholded and transformed.
![alt text][image11]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Using a histogram to define where lane pixels are condensed, and using the sliding window algorithm introduced in the lectures, i implemented a function that would take an image, and execute the algorithm on it, where it would define the lane pixels and then fit a second-order polynomial describing the lanes. To decrease computation, the output of the sliding window algorithm ie: Lane indices and polynomial coefficants, were used as a starting point and as a search margin for a faster function, that instead of searching the whole image for lane lines, started off with what the original function outputted and a margin. Function name is `sliding_window_polyfit()`

![alt text][image14]

Sliding window algorithm with a search margin. Function name is `polyfit_using_prev_fit()`

![alt text][image15]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Again, using the code snippet provided in the classroom, a method was built upon it, where it calculated the curvature of the polynomials using a mathematical formula explained in the lectures. Function name is `curv()`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

A function with the name `draw_lane()` based on the snippet provided in the lectures was constructed, where it would super impose a colored lane on top of the real lane by warping it using the ivnerse calibration matrix and correction coefficients. It would basically colour between the lines.

![alt text][image17]

Data regarding curvature and center distance were superimposed on the frames.

![alt text][image18]


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


It took me a long time experimenting with different types of gradients, color spaces, and thresholds. With the help of the forum i was able to understand more about the effects of each choice and reach a resonably robust thresholding function for the project video. Understanding how to use the result of fitting through each stage in the processing and using them in sequentially was hard to understand for a while but i got around to it. At some stages of the video, lane lines could not be identified and my algorithms fail, and caused the processing to stop, so i averaged a number of previous lines to figure out where the undetected lanes, because logically it would not just magically turn 90 degrees over. A more robust thresholding technique, and better sanity check would definietely better the pipeline, and i will be working this project again, after i am done with the deadlines, as it is rich in material and experimentation
