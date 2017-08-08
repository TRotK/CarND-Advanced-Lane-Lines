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

[image1]: ./output_images/Undistortion.png "Undistorted"
[image2]: ./output_images/undist_test1.png "Road Transformed"
[image3]: ./output_images/Bin_test3.png "Binary Example"
[image4]: ./output_images/PT_straight_lines1.png "Warp Example"
[image5]: ./output_images/search.png "Fit Visual"
[image6]: ./output_images/map_lanes.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z = 0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I first load the pre-computed calibration matrix and distortion coefficients, then using `cv2.undistort()` to perform distortion correction an test image. The result is shown below.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (saturation) and gradient thresholds to generate a binary image (thresholding steps in the code cells under Binarization section). I also applied a Gaussian filter with kernel size of 5 to depress noise. Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in in the code cells under Perspective Transform section. The `warper()` function takes as inputs an image (`img`), as well as the Homography H computed from source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 64, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 64), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 576, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 704, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing a polygon connecting the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did histogram and sliding window search to find lane pixels, and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

In video test, I also did temporal averaging on the last 10 frames to smooth the lane detection.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Calculating the radius of curvature is done using the formula with the estimated polynomial coefficients and the bottom y value. The vehicle position with respect to the center of lanes is done by calculating the difference between the camera center in x and the lane center. I then converted them both from the pixel to world space.

I did this in the cell 3 through 5 under the under Lane Detection section

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the last two code cells under the Lane Detection section in the function `markLanes()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My implemetion of lane detection is a bit jittered for those road parts whose paving is bright and lanes are unclear. Temporal averaging helped a lot. Further exploration is using outlier rejection like RANSAC to keep the real lane pixels in the above situation (as I implemented in the first project).

In the challenge_video.mp4, the main difficulties are that there is a disruption in the middle of lanes and is repaved with different color, and the left lane is too close to the road side which makes the detector confisued about which one is the lane. So, a possible solution is to weight more on color as the lane marks are always in yellow or white.

For the harder_challenge_video.mp4, I feel it would be really hard to detect the lanes by purely apply the computer vision techniques taught in class. Perhaps, using deep learning to segment lanes region would help. Please give me some inspiration if possible.
