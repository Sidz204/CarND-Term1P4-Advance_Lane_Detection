## Advance Lane Detection

### In this project, we need to write a software pipeline to identify the lane boundaries in a video.
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


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Code in cells :3 and 4 in ipynb file
I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Here is an image of corners drawn on chessboard:

![calibrate](/output_images/calibrate.png)

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Original image :

![original](/test_images/straight_lines1.jpg)


Undistorted image:

![original](/output_images/undist.png)


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The matrix and coefficients were then used to undistort any given image taken from the camera.To demonstrate this step, Its explained in the above picture how a original image looks and how a distortion corrected image.


#### 2. Color transforms, gradients or other methods to create a thresholded binary image.

I used a combination of color(S and L channels of HSL) and gradient thresholds to generate a binary image (thresholding steps in cell :7  in `advance_lane.ipynb`). Through experimentation I found out some perfect threshold values.Finally, I used bitwise AND to combine S and L channel thresholds and used bitwise OR to combine them further with sobelxy gradient.Here's an example of my output for this step.

![binary](/output_images/binary.png)

#### 3.Performing Perspective transform (Bird's eye view of image).

The code for my perspective transform includes a function called `warp_perspective()`, which appears in cell 6 in the file `advance_lane.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as M (transformation matrix) which is calculated using source (`src`) and destination (`dst`) points. Through experimentations I found some good src and dst points as follows:


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 560, 460      | 120, 0        | 
| 100, 700      | 80, 700       |
| 735, 460      | 980, 0        |
| 1120, 700     | 1100, 700     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![perspective](/output_images/warped.png)

#### 4. Identify lane-line pixels and fit their positions with a polynomial

I then began my procedure for finding the lane lines in the image, with the code provided in the classroom as my starting point. The function find_path() is used to detect lane lines in the image. It takes as input a binary transformed image and identifies the lanes. The code begins by drawing a histogram of the image. Since by now we have a binary image, with reduced noise, the lanes should be the highest peaks in the histogram. Using this we identify the mid-point of the image. Points upto the the mid-point are classifed as left and those greater than the mid-point are considered right. A sliding window search is then employed and the non-zero values obtained within this region was obtained. The points to the left were saved as good left indices and the points to the right of midpoint were saved as good right indices. These points were then plotted in the arrays left_fitx and right_fitx. And then finally, I fit my lane lines with a 2nd order polynomial kinda like this:

![boxes](/output_images/boxes.png)


#### 5. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

For the radius of curvature, a polynomial equation was fitted through the location points of the lanes obtain to form a polynomial in y. The radius was obtained using a formula R = ((1 + (dx/dy)^2)^(3/2))/(d2x/dy2). Code provided in the measuring curvature part of classroom material served as help and position was measured comparing the center of the image i.e. 640. I finally printed radius and position at top left corner in FONT_HERSHEY_SIMPLEX font.


#### 6. Example image of the result

I implemented this step in the function `find_path()`.  Using src and dst points, I calculated Minv i.e. inverse transformation matrix to unwarp the original image back to normal view. Here is an example of my result on a test image:

![boxes](/output_images/final.png)


---

### Pipeline (video)

#### 1. Here's a link to my final video output.

Here's a [link to my video result](./final.mp4)

---

### Discussion

#### 1. Problems / issues you faced or drawbacks in your implementation of this project.

- I have not used the technique of averaging the lines and using lines detected with previous frames to get the output of current frame. I am working on it.
- Also, I have hardcoded the source and the destination points which can be made dynamic helping to make our pipeline more robust.
- If any obstacle like a bike in harder challenge video, the detection will change. Need to identify some technique so that it still detects the lane even if there are obstacles.
