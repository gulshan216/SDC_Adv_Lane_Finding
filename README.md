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

[image1]: ./output_images/Undistorted.png "Undistorted"
[image2]: ./output_images/Test_img_undistorted.png "Road Transformed"
[image3]: ./output_images/thresholded_img.png "Binary Example"
[image4]: ./output_images/Warp_straight_lines.png "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./output_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration


The code for this step is contained in the first code cell of the IPython notebook located in `Adv_Lane_Finding.ipynb` .  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_pts` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_pts` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_pts` and `img_pts` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Using color transforms, gradients to create a thresholded binary image.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in fourth code cell in of the IPython notebook).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Performing a perspective transform on the thresholded binary image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears sixth code cell of the IPython notebook .I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(image.shape[1]/2)-405,image.shape[0]-50],
    [(image.shape[1]/2)-55,(image.shape[0]/2)+90],
    [(image.shape[1]/2)+60,(image.shape[0]/2)+90],
    [(image.shape[1]/2)+470,image.shape[0]-50]])
offset = image.shape[1]*0.25
dst = np.float32(
    [[offset,image.shape[0]],
    [offset,0],
    [image.shape[1]-offset,0],
    [image.shape[1]-offset,image.shape[0]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 235, 670      | 320, 720      | 
| 585, 450      | 320, 0        |
| 700, 450      | 960, 0        |
| 1100, 670     | 960, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identifying lane-line pixels and fit their positions with a polynomial

Then I basically performed the sliding window search to search for left and right lane line points in my warped image.You can see this seventh code cell in my Ipython notebook. I chose 9 equally sized windows with a width of 100 pixels. I also take a minimum threshold of 50 pixels to be found in the window before the lane line center points are changed. Then I perform smoothing over the last 12 samples to find the best fit left and right lane points. Then I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Calculating the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in eighth cell of my Ipython notebook. After finding the lane lines in pixel space i now try to find the lane lines in the real world space. To do this i first measure the lane width and the length of the dashed lane lines in my warped image in terms of pixels. This turns out to be 554 and 110 pixels respectively. Since the minimum lane width according to U.S regulations is 3.7 meters and and the dashed lane lines are 10 feet or 3 meters long each. I chose my conversion factor for my x dimesion to be 3.7/554 and for y dimension to be 3/110. Then i fit a curve after transforming the lane points in my warped image to real world points using the conversion factor. Then i perform the calculation of the radius of curvature as given in this [link](https://www.intmath.com/applications-differentiation/8-radius-curvature.php).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the eighth cell of my Ipython notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.

* Using the color thresholding really helped in picking the lane lines. Without color thresholding it was difficult to pick lane lines in varying lighting conditions. 
Hence I used the S channel in the HLS color space for picking out the lane lines. I also combined it with a threshold of the V channel from the HSV color space as it helped me remove the shadows that were being detected by the s channel.
* Initially without adding the smoothing of the previous samples, the lane lines detected in the video were quite wobbly. Hence adding smoothing over the last 12 samples helped in detecting the lane lines smoothly and calculating the right lane curvature.    
