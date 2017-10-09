**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Use color transforms, gradients, etc., to create a thresholded binary image for binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/test1.png "Road Transformed"
[image3]: ./output_images/bird_view.png "Bird View Example"
[image4]: ./output_images/binary_bird_view.png "Binary Bird View Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./test_videos_output/project_video.mp4 "Video"
[video2]: ./test_videos_output/challenge_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the `Image_processing.calibration` function and `Image_processing.undistort` function in `Define Image procesing() class` code cell and `Calibrate camera with calibration images` code cell of the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the `undistort()` function in `Define Image procesing() class` code cell of the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb".  

Based on the the camera matrix and distortion coefficients from Camera Calibration, I applied the distortion correction to one of the test images like this one:

![alt text][image2]


#### 2. Describe how I performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes in a function called `birds_eye()`, which appears in `Define Image procesing() class` code cell in the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb".  The `birds_eye()` function takes as inputs a distortion-corrected image (`undistorted_img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
lane_shape = [(584, 458), (701, 458), (295, 665), (1022, 665)]
top_left, top_right, bottom_left, bottom_right = lane_shape
self.src = np.float32([top_left, top_right, bottom_right, bottom_left])
self.dst = np.float32([[self.img_width/4,0], [self.img_width*3/4,0],
                       [self.img_width*3/4,self.img_height-1], 
                       [self.img_width/4, self.img_height-1]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 584, 458      | 320, 0        | 
| 701, 458      | 960, 0        |
| 295, 665      | 960, 719      |
| 1022, 665     | 320, 719      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

#### 3. Describe how I used color transforms, gradients or other methods to create a thresholded binary bird-view image.  Provide an example of a binary image result.

I used a combination of gradient thresholds (gradient in x direction, gradient in y direction and magnitude of the gradient) and color thresholds (S channel of HLS color space, B channel of LAB color space, V channel of HSV color space and L channel of HLS color space) to generate a binary image (the code for thresholding includes a function called `score_pixles()`, which appears in `Define Image procesing() class` code cell in the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb").  Here's an example of my output for this step.  

![alt text][image4]


#### 4. Describe how I identified lane-line pixels and fit their positions with a polynomial?

Then I find the starting point of each lane with the method of taking histogram of the bottom half of the image. Based on sliding windows, I identified the nonzero pixels in the binary image to each lane. And fit my lane lines with a 2nd order polynomial kinda like this (note: this is not actually from one of the test images):

![alt text][image5]

The pixels of new lanes in a new binary image can be identified with the identified lines in the previous frame of video. 

The code for identifying lanes includes in a function called `Process_image()`, which appears in `Define Process_image() class` code cell in the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb".  The `Process_image()` function takes as inputs an original image (`image`).

 
#### 5. Describe how I calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature of the lane in a function called `update_curvature()`, which appears in `Define Line() class` code cell in the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb". And I calculated the position of the vehicle with respect to center in a function called `get_position_from_center()`, which appears in `Define Line() class` code cell in the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb".

#### 6. Provide an example image of my result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `6. Provide an example image of your result plotted back down onto the road.` code cell in the IPython notebook located in "./examples/AdvanceLaneFinding.ipynb".  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to my final video outputs.  

Here's a [link to my project video result](./test_videos_output/project_video.mp4)

Here's a [link to my challenge video result](./test_videos_output/challenge_video.mp4)

---

### Discussion

1. The most challenge task is the design of the combinations of different methods to create a thresholded binary bird-view image. The gradient thresholds based methods are good at locating the white dash lane (the right lane in the project), but also easily affected by shadow and dark asphalt.  The color thresholds based methods are very solid to detect the yellow solid lane (the left lane in the project), but totally failed to detect the dash lane. The combination of them increased the capability of locating the lanes in the project video and challenge video. However, it is not robust to the shadow, shine and debris aside the driving lane in harder challenge video. 
2. The lane finding algorithm works well for both project video and challenge video, however, it fails for harder challenge video. Since the lane finding algorithm limits the direction variation rate, the algorithm cannot provide correct result if the direction of the lane changes quickly, which is the case in harder challenge video, and/or the initial lane detection is not correct. 
3. The assumption does not hold that both lanes reach the top and bottom of the bird-view image. It is another reason the land finding algorithm fails in harder challenge video.
4. To make the pipeline more robust, a score for each pixel in the image should be implemented to rate if the pixel is in lane. And an advanced lane finding algorithm should be designed. The advanced algorithm can estimate the coefficients of each lane with less pixel found in the binary image, check both lanes are roughly parallel and adapt the direction change rate in the cases of sharp turn.