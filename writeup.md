# **Advanced Lane Finding Project**
***
My whole work could be found under the `Advanced Lane Finding Project.ipynb` and
this writeup summarizes the results.

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

[image1]: ./img/chessboard-drawn.png "Chessboard Drawn"
[image2]: ./img/calibration-images-undistorted.png "Calibration Images Undistorted"
[image3]: ./img/test-images-undistorted.png "Test Images Undistorted"
[image4]: ./img/undistortion-comparison.png "Comparing Undistorted Image"
[image5]: ./img/thresholded-images.png "Test Images Binary Thresholds"
[image6]: ./img/perspective-transform-images.png "Warped Binary Images"
[image7]: ./img/perspective-transform-comparison.png "Comparing Perspective Transform"
[image8]: ./img/sliding-windows-images.png "Sliding Windows Images"
[image9]: ./img/road-images.png "Road Images"
[image10]: ./img/road-with-infos.png "Road Images with Information Annotated"
[image11]: ./img/unwarped-images.png "Unwarped Lane onto Original Images"
***

## Writeup / README

You're reading it!

## Camera Calibration

The work for this rubric could be found in the notebook under the part with heading
`1. Camera Calibration and Test Images undistortion`.

It starts with collecting the coordinates of the chessboard corners using the
`cv2.findChessboardCorners()` and append them to the `objpoints` and `imgpoints`
iterables, to be passed onto the `calibrate_camera()` function I created in the
`Helper Functions` part of the notebook.

When parsing the images, I drew the corners on them and output them under the folder
`output_images/chessboard_drawn`. Here is the overview of them:
![Images of chessboard with drawn corners][image1]

## Pipeline (test images)

### 1. Provide an example of a distortion-corrected image.
The work for this rubric could also be found in the notebook under the part with
heading `1. Camera Calibration and Test Images undistortion`.

The iterables defined previously are then passed to the `calibrate_camera()` function
which basically uses the `cv2.calibrateCamera()` to obtain the **camera matrix**
`mtx` and the **distortion coefficients** `dist`.

With this, I undistorted all the calibration and test images with the `undistort()`
function, defined in the `Helper Functions` part of the notebook. Here is the
overview of the result:
![Calibration Images Undistorted][image2]
![Test Images Undistorted][image3]

And here is a side by side comparison of the calibration images with a clear
difference:
![Undistortion Comparison Image][image4]

Those images are also saved under the directory `./output_images/undistorted`.

### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this rubric is available in the notebook under the part with a heading
`2. Color and Gradient Threshold`.

I use the `combined_threshold()` function to threshold the images. It is a combination
of Sobel X filtering with Saturation Color threshold with the following parameters:
- Kernel Size: 15
- Sobel X Threshold: (20, 60)
- HLS Threshold: (170, 250)

Here is the overview of the outputs:
![Thresholded Images][image5]

The outputs are also saved under the directory `./output_images/color_gradient_threshold`.

### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this rubric could be found in the notebook under the part with the
heading `3. Perspective Transform`.

I basically use some helper functions such as `getTransformMatrix()` and `transformPerspective()`, defined in the `Helper Functions` part of the notebook,
which utilize `cv2.getPerspectiveTransform()` and `cv2.warpPerspective()`.

The parameters I used as the source and destination coordinates are the following:
```python
x_src = [215, 597, 687, 1105]
y_src = [720, 450, 450, 720]
src_coors = np.float32([(x_src[0],y_src[0]),\
                        (x_src[1],y_src[1]),\
                        (x_src[2],y_src[2]),\
                        (x_src[3],y_src[3])])
dst_coors = np.float32([(290, y_src[0] ),\
                        (290, 0),\
                        (990, 0),\
                        (990, y_src[3])])
```

This results in a perspective transform such as the following:
![Perspective Transform Comparison][image7]

Here is also an overview of the output of this step to the binary thersholded test
images:
![Warped Binary Images][image6]

The outputs could also be found under the directory `./output_images/binary_warped`.

### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
The work for this rubric could be find in the notebook under the part with heading
`4. Lane Finding`.

So, I use the following helper functions defined in the `Helper Functions`:
- For drawing on the images:
  - `draw_window()` at `line 214`
  - `draw_road()` at `line 238`
  - `draw_lane_px()` at `line 264`
  - `draw_fit_lines()` at `line 284`
- To find lanes:
  - `sw_find_lane_with_window()` at `line 309`
  - `sw_find_lane_pixel()` at `line 413`
  - `search_around_poly()` at `line 566`
- To find fitted lines polynomial
  -  `fit_poly()` at `line 506`

All of the helper functions above utilize a `Line()` class and its children, defined
under the part in the notebook with heading `4.0. Creating Line Classes to Help in Video Processing`. I will elaborate more regarding this classes below on the rubric for
the video processing.

The main results are under the heading `4.1. Applying Sliding Windows on Test Images`
and `4.3. Applying Sliding Windows with Road Drawn on Test Images`.

The general overview of what I did in this step is the following:
1. Input undistorted test images
2. Apply gradient and color threshold
3. Apply perspective transform
4. Apply sliding windows or search around poly to find lanes
5. Draw lane lines and the road

Here is the overview of the output of the sliding windows lane finding:
![Sliding Windows Images][image8]

And here is the overview of the output of the sliding windows with road also drawn:
![Sliding Windows with Road Images][image9]

The outputs could also be found under the following directories:
- `./output_images/sliding_windows` for the output images with windows drawn
- `./output_images/sliding_windows_lane` for the output imagages with no window drawn
- `./output_images/lane_found` for the output images of sliding windows, where the lane lines and the road drawn
- `./output_images/prior_poly` for the output images of search around poly

### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The work for this rubric is in the notebook under the part with a heading
`5. Curvature and Position Finding`.

The main work of the curvature finding and the vehicle position with respect to
the center are by the helper functions `measure_curvature_real()` and
`vehicle_offset()`, which could be found under the `Helper Functions` part in the
notebook at the `line 651` and `line 684`.

I also use a helper function `draw_curv_pos_text()` under the `Helper Function`
part at `line 615` to draw those informations onto the image.

Here is the overview of the outputs:
![Curvature and Position Images][image10]

The outputs could also be found under the directory `./output_images/curvature_position`.

### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this rubric is found in the notebook under the part with a heading
`6. Lane Unwarp`.

Basically, I only find the inverse matrix for the inverse perspective transformation
of the lane and road found image, and then overlay it onto the original image.
Here's my code for them:

```python
# To obtain the inverse matrix, simply swap the src coordinates with the dst
Minv = getTransformMatrix(img, dst_coors, src_coors)

# Transform the warped image
inverse_lane = transformPerspective(road_lane_img, Minv)

# Overlay the warped image onto the original
overlaid_img = cv2.addWeighted(img, 1, inverse_lane, 1, 0)
```

Here is the overview of the outputs:
![Unwarped Images][image11]

***

## Pipeline (video)

### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The code for this rubric could be found in the notebook under the part with the
heading `7. Full Pipeline for the Videos` and `4.0. Creating Line Classes to Help in Video Processing`.

Here are the overview of the algorithm:
1. Undistort the image
2. Gradient and color threshold application
3. Transform to bird's eye view
4. If `Line.bestx` is found and search around poly has **not failed for more than 5 times**:
    * Run **search around poly** flow
    * Check Sanity
    * If sane:
      * Fail count is reseted to 0
      * Append current X coordinates fitted to class variable
    * If not:
      * Iterate fail count
      * Draw lines from `self.bestx` (best fitted from previous lines)
5. If no `Line.bestx`:
    * Run **sliding windows** flow
    * Check Sanity
    * If sane:
      * Fail count is reseted to 0
      * Append current X coordinates fitted to class variable
    * If not:
      * Iterate fail count
      * Draw lanes anyway

The `sanity_check()` could be found under the `7. Full Pipeline for the Videos`,
of which I define sanity as having more or less the same curvature of the left
and right lines and having their distance around 2.9 and 4.5 meters.

The class `Lane()` is adopted verbatim from the class, and added a property
`self.current_fitx` to save the fitted X coordinates of the lane lines.

I then created two children `LeftLine()` and `RightLine()` to store X fitted
coordinates of the prior left and right lines in a class variable `prev_xfitted`.
Besides that, I also create class variable named `fail_count` as a counter to
how many times the search around poly fail to find sane lines.

On their instantiation, I append the `prev_xfitted` to the `self.recent_xfitted`
and calculate the `self.bestx` and `self.best_fit` from the maximum of previous
three sane lines. This is later to be used for the search around poly flow.

The output videos could be found under the `./output_videos` directory.

**IMPORTANT NOTE:**
I need to always rerun the class definition every time I process the video, as
to reset the class variable within it, so it does not contain the previous lines
from the frame of the previously processed video.

***

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

- When the road color changes, it sometimes pose some problem to the lane finding algorithm as is. This might be improved on by applying different gradient and color threshold combinations under different road lightings: dark, well lit, etc.
- I couldn't tackle the challenge videos with my current code as is. The problem that I found is that there are long patches on the road that are detected to be the lane lines. This problem might be alleviated by integrating an upgraded masking area from the previous project, and only selecting area where only the left or right lanes could be within, not outside or between.
- From the harder challenge video, I learned that there might be a problem where only one lane line is detected (left or right). This might be alleviated by extrapolating the other missing line from the one detected.
