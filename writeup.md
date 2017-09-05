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
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[image7]: ./output_images/undistort-test5.jpg "undistort-test5"
[image8]: ./test_images/test5.jpg "test5"
[image9]: ./output_images/warped-test5.jpg "warp"
[image10]: ./output_images/recover-test5.jpg "recover"
[image11]: ./output_images/color_filter.jpg "color_filter"
[image12]: ./output_images/lane_fit-test5.jpg "lane fit"
[image13]: ./output_images/plottest5.jpg "recovered test5"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./advanced_lane_lines.ipynb" first cell

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image 5 using the `cv2.undistort()` function and obtained this result: 

![undistort result of test5][image7]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![test5][image8]
I applied camera caliberation results from previous cell and apply to "test5" image using `cv2.undistort`, here is result I got

![undistort result of test5][image7]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I mainly use color threshold to generate binary image, where I tried to filter out non yellow and white on HSV color space. Alternatively, I choose to apply sobel based filter. Finally, the color based filter offers better result.

![undistort test5 after color filter white and yellow][image11]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in function `get_bird_eye_binary_wrap` (in the 2nd code cell of the IPython notebook).  The `get_bird_eye_binary_wrap()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![bird eye image][image9]
![recover image][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![lane fit][image12]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this cell 6 where I copied from part 35 of material provided by lecture. It basiclly try to caculate lane fit basec on 2nd order polynomical function f(y)=Ay2+By+C and apply curvature function described http://www.intmath.com/applications-differentiation/8-radius-curvature.php

The result of left lane fit is 2855.44424885 m and right lane fit is 1664.59075647 m of test5


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 7 where I implemented `overlay` funciton, it draw left and right fit 2nd polynomical line with fill polyfill as Green
and then it overlay on top of original image with 30% of weight via `addWeighted`

![overlayed lane detection to test 5][image13]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I spent fair amount of time to determine if I should go HSL plus sobel filter approach or HSV and color based filter approach. I started with HSL and sobel approach as recommended in text, the result seems not quite satisfactory where the poly fit gives a drifted overlay on top of actual lane lines. So I started to use color based approach which assume yellow and white color filter works better. It works most of time as well as all six test images. The problem kicks in when there are cases where video may not find left and right lane fits. I decided to go for a primitive approach which takes previous found left and right fits curves and keep what it is if current frame can't update new values.
