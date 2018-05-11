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
[image5]: ./output_images/test2_lanefinding.jpg "Fit Visual"
[image6]: ./output_images/lane_remap.jpg "Output"
[image7]: ./mess1.jpg "mess"
[image8]: ./mess2.jpg "mess"
[image9]: ./mess3.jpg "mess"
[image10]: ./mess4.jpg "mess"

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

After getting my threholded warped image, I use the "Peaks in a Histogram" method to find the lanes. Specifically, I first take a histogram along all the columns in the lower half of the image. Then I am using a sliding window method to search for lines where the most pixels in binary image locate. The left and right lane search bases are the two peaks in histogram. After all the good pixel points are identified for left and right line, then I use the `np.polyfit` function to fit a polynomial along those points. All the code for this part are in the 8th code cell of the IPython notebook [Project4Pipeline_simplified.ipynb](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/Project4Pipeline_simplified.ipynb).

 One example result for [test2.jpg](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/test_images/test2.jpg) is shown here:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Since I get the polynomial fit for the left and right lane, we can use that to calculate the radius of curvature based on the equation of radius of curvature. Here I calculate the curve radius at the bottom point of left and right lane line since it is closest to the vehicle. After I get the radius of curvature for left and right lane, I take average of them to get the curve radius of the lane. Then the last step is to convert the pixel curve radius into real curve radius in meter, this is based on the conversions in x and y from pixels space to meters:

`
ym_per_pix = 30/720
xm_per_pix =3.7/700
`

For the position of the vehicle with respect to lane center, I assume the center of the vehicle is the same of the center of camera image and the lane center is the midpoint at the bottom of the image between left and right lane lines. The different between image center the midpoint of left and right lane line is the vehicle position with respect to lane center. If the offset of vehicle center minus lane center is position then it is identified as right side from lane center, otherwise it is identified as on left side of lane center.

The code for this part is covered in the last part of 8th code cell in the IPython notebook [Project4Pipeline_simplified.ipynb](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/Project4Pipeline_simplified.ipynb).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 9th code cell of the IPython notebook [Project4Pipeline_simplified.ipynb](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/Project4Pipeline_simplified.ipynb) in the function `lane_drawing()`.  Here is an example of my result on a [test2.jpg](https://github.com/GitHubChuanYu/Project4_AdvancedLaneFinding/blob/master/test_images/test2.jpg) image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One interesting problem I have faced in my implementation is when I use this color and gradient threshold combination:
`combined_binary[((binary_H == 1) & (binary_S == 1)) | (gradx == 1)] = 1`

for a specific image in the video when the lane change from white paved to dark paved. Then the result is totally messed up like what I have shown here:

![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]

After analyzing it, I think the main reason is there are a lot of noisy pixels (from gradient part) between left and right lane due to the road brightness change from light to dark. And the histogram shows that the peak for right lane bottom point is misdetected. So the right lane fitting polynomial is messed up for right lane. This could be improved by setting the histogram to include all the whole image instead of bottom half for peak points. However I have tried that and it turns that that it solved this problem but cause another misdection of peak point due the noisy pixel on top part of the image. So finally I use the current combined threshold method which is purely color threhold comination of L(HLS) and B(LAB). It can filter a lot of noisy pixels between left and right lane.

However, I think my current method could also have some limitation due to strong capability of filtering noisy pixels, for example if one of the lanes is very small and dark, then it would also be filtered and then the problem is this lane is not detected.

I think to increase robustness of my current lane finding pipeline, I can implement the look-ahead filter which can utilize the previous lane finding polynomial to increase robustness of adjacent future image. But due to limited time and capability on Python coding, I have not tried that implementation.
