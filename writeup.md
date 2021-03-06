

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

[image1]: ./output_images/undistort_chessboard.png "Undistorted"
[image2]: ./output_images/undistort.png "Road Transformed"
[image3]: ./output_images/HLS_transform.png "HLS Transform"
[image4]: ./output_images/binary_image.png "Binary Image"
[image5]: ./output_images/warped_image.png "Warp Example"
[image6]: ./output_images/histogram "Histogram Example"
[image7]: ./output_images/color_fit_lines.png "Fit Visual"
[image8]: ./output_images/example_output.png "Output"
[video1]: ./result_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I transformed HLS image from RGB image, and among the three channels, S channel was used. I only used the S channel because I could clearly distinguish the lanes from the shaded areas. Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 15th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as the mapping matrix which is the result of cv2.getPerspectiveTransform. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 40, img_size[1] / 2 + 100],
     [((img_size[0] / 6) + 80), img_size[1]],
     [(img_size[0] * 5 / 6) + 80, img_size[1]],
     [(img_size[0] / 2 + 80), img_size[1] / 2 + 100]])
  
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 600, 460      | 320, 0        | 
| 293.33, 720   | 320, 720      |
| 1146.67, 720  | 960, 720      |
| 720, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I take a histogram along all the columns in the lower half and here is the result of histogram.
![alt text][image6]
I found the peak of the left and right halves of the histogram, and these were used for the starting point for the left and right lines. From the starting points, I slided ( height/9 x 100 ) size of window and identified non-zero pixels in the window. Finally, I fit my lane lines with a 2nd order polynomial kinda like this:
![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function is implemented as the RadiusMeasuringInMeters function in the Line class.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 27th code cell of the IPython notebook. Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

When applying the algorithm for the first time, there was a case that the lane was misfound when the color of the road changed or the shade occurred. First, the valid lane width, camera location, and lane radius are used to identify if the lane found is incorrect. Second, I used averaging over time axes for more robust results. These two processes have produced good results for the difficult cases I was concerned about.
