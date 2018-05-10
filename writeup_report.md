## Writeup Report

### This is Chuan's writeup report file for Udacity Self-Driving Car nanodegree term1 project 4 Advanced Lane Finding

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

[image1]: ./output_images/calibration15.jpg "ChessboardWithCorners"
[image2]: ./output_images/undistort.jpg "Undistort effect"
[image3]: ./output_images/test2_thresheld.jpg "Binary Example"
[image4]: ./output_images/straightlinelane_PT_1.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how I addressed each one. 

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the IPython notebook located in "Project4Pipeline_simplified.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  An example image showing original chessboard image with img points on it is:

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image test4.jpg using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of L channel in HLS color space and B channel in LAB color space thresholds to generate a binary image. The code is shown as:

```python
binary_H = HLS_Hthresh(undistorted, thresh=(10,100))
binary_S = HLS_Sthresh(undistorted, thresh=(170,255))
binary_L = HLS_Lthresh(undistorted, thresh=(220,255))
binary_B = LAB_Bthresh(undistorted, thresh=(155,255))
gradx = abs_sobel_thresh(undistorted, orient='x', sobel_kernel=3, thresh=(15, 100))
combined_binary = np.zeros_like(binary_L)
combined_binary[(binary_L == 1) | (binary_B == 1)] = 1
```

Here's an example of my output from [test2.jpg](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/test_images/test2.jpg) for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the 7th code cell of the IPython notebook [Project4Pipeline_simplified.ipynb](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/Project4Pipeline_simplified.ipynb).  The `warp()` function takes as inputs an image (`img`), and the source (`src`) and destination (`dst`) points are calculated inside the function.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[576.651, 462.127],
    [235.583, 700.472],
    [1066.41, 700.42],
    [707.714, 462.127]])
dst = np.float32(
    [[(img_size[0] / 4) - 70, 0],
    [(img_size[0] / 4) - 70, img_size[1]],
    [(img_size[0] * 3 / 4) + 70, img_size[1]],
    [(img_size[0] * 3 / 4) + 70, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 577, 462      | 250, 0        | 
| 236, 700      | 250, 720      |
| 1066, 700     | 1030, 720      |
| 708, 462      | 1030, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image with relatively straight line lane [straight_lines2.jpg](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/test_images/straight_lines2.jpg) and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
