# **Advanced Lane Finding Project**

The goals of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./intermediate_steps/chess_marked.jpg "Undistorted"
[image2]: ./intermediate_steps/sobel_thresh.jpg "Road Transformed"
[image3]: ./intermediate_steps/combined_thresh.jpg "Binary Example"
[image4]: ./intermediate_steps/transformed.jpg "Warp Example"
[image5]: ./intermediate_steps/polyfit.jpg "Fit Visual"
[image6]: ./intermediate_steps/example_output.jpg "Output"
[video1]: ./video_output/project_video.mp4 "Video"

## Final result
The video of the final result can be downloaded [here](./video_output/project_video.mp4)

## How To View This Project
1. Install jupyter notebook from [here](https://jupyter.org/install)
2. Navigate to the folder in your file system and run `jupyter notebook`
3. Open 'LaneFinding.ipynb'
4. Click 'Run All'

## How the Project Meets the Goals

### Camera Calibration
This is achieved in cell #2 of the jupyter notebook.

The camera was used to take a picture of a chess pattern at many different angles. The chess pattern is assumed to be at a fixed point on the (x,y) plane at z=0. Consequently, the object points of each corner in the pattern are the same for each image. For each image, the set of image points and their corresponding object points are compiled. This is then utilised in an openCV function to calculate the camera matrix, and correction coefficients.

The image resulting from the use of these functions is free from axial and tangential distortions. The below image is an example of a calibrated image. This process of identifying the coefficients and matrices only needs to be conducted once. Each image calibration can be done using the same values.

![Distortion Free Image][image1]

### Pipeline Process
#### Identifying the lane
After undistorting the image, the first step is to utlise a variety of image features to detect components of the frame that could be the lane. I used a combination of sobel gradients, and colour transforms in order to achieve this.

The sobel component of the analysis uses the directional gradient of the grayscale image's pixel intensity to detect lines in certain directions. `gradx` gives the sobel gradient in the x direction, and `grady` in the y direction. Additionaly, the direction of greatest gradient `dir_binary` at each pixel was also compared with the overall gradient magnitude `mag_sobel`. These four parameters used  predetermined thresholds to determine how sensitive they were to the image. The resulting comparison between these four parameters is shown below

```
((gradx == 1) & (grady == 1)) | ((mag_sobel == 1) & (dir_sobel == 1)) = 1
```
The final combined sobel threshold image is shown below.

![alt text][image2]

The colour component of the analysis was achieved by converting the image into the HLS colour encoding format. This is due to the unique property that the saturation of a colour is fairly constant regardless of lighting conditions. Thus this saturation channel was used as a major filter for lane lines. This can be seen below, and in the jupyter notebook.

![alt text][image3]

#### Perspective Transform

The code for my perspective transform includes a function called `transform_perspective()`, which appears in the 7th code cell of the IPython notebook).  The function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points based on an image of a straight road in the following manner:

```python
    src = np.float32([[225,700],[1080,700],[594,450], [685,450]])
    dst = np.float32([[300,y], [x-300, y], [300, 0], [x-300, 0]])
```
After much tuning, these points resulted in a perfect perspective transform for all images. The resulting lane lines are consistetly parallel, and perfeclty straight (if the road is also straight). I this by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. Find below an example below.

![alt text][image4]

#### Lane Identification and Fitting
Using this transformed image, the lanes were to be detected. This was achieved by using the histogram and sliding window method. A histogram was used to identify the regions on the left and right side of the image with the most amount of positive identifications. Consequently, these were estimated to be the lanes. A series of small windows were subsequently fitted over these suspected lanes to determine a which points are a part of the background, and which are a part of the actual road.

From these list of points, a 2nd order polynomial was fit to model the curvature of the lane. This polynomial fit was then transformed back into the perspective of the original image and mapped onto the screen.

![alt text][image5]

#### Curvature and Offset
The radius of the car's turn is calculated by using the curvature of the polynomial fit at the bottom of the screen (closest to the car). Once the curvature is mathematically formulated, the model must be shifted from the image space into the real world. This is achieved by assuming that the road is a flat plane, and that each pixel in the horizontal and vertical direction maps to a fixed distance in the real world. These approximations are shown below and the resulting polynomial fit is turned into a real-world result by utilising these correction factors.

```
ym_per_pix = 30/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension
```
To calculate the 'offset' of the car from the middle of the lane, the video is assumed to be taken from the centre of the car. The distance from the centre of the image to each polynomial fit is thus measured, and multiplied by the correction factors above. The offset is half of the difference between these two distances.

#### Computational Complexity Reduction

To reduce the computational complexity of the pipeline, once the trend-line has been determined via the histogram and window method, the future frames are mapped by simply using the pixels nearest to the previous fit. This prevents the entire image needing to be scanned in every single frame.

---
### Reflection

This pipeline works well for straight and reasonably curved roads, as well as in various light conditions. Unfortunately, it does not do very well when additional lines are on the road. For instance, if two halves of the lane are paved in different colours, the pipeline will identify the middle region as a lane.

To improve this pipeline, it would be beneficial to incorporate a machine learning model that can identify which line is a lane, and which is an environmental anomaly.

In addition, if the road is curved too far in one direction, the histograms will continue to stack up, and will not identify that the lane has left the view point. As a result, the fit is inaccurate for curved roads. This can be mitigated by modelling the lane progressively, as the windows are added.
