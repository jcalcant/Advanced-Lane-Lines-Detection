# **Advanced Lane Finding Project**


**Advanced Lane Finding Project**  
The goals / steps of this project are the following:
* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle
position.

[//]: # (Image References)

[corners]: ./output_images/corners_found5.jpg
[color_filters]: ./output_images/color_filters.png
[gradient_dir]: ./output_images/grad_dir_threshold.png
[canny_edges]: ./output_images/canny_edges.png
[masked_image]: ./output_images/masked_image.png
[masked_edges]: ./output_images/masked_edges.png
[warped_image]: ./output_images/warped_image.png
[polynomial]: ./output_images/polynomial.png
[lane]: ./output_images/lane.png
[original]: ./output_images/original.png
[undistorted]: ./output_images/undistorted.png
[original_cal]: ./output_images/original_cal.png
[undist_cal]: ./output_images/undist_cal.png

---
## 1. Methods

### Camera calibration

With the chessboard images provided, I calibrated the camera in the following way to correct the radial and tangential distortions from the camera:\
First, I converted the images to grayscale. Then, I applied cv2.findChessboardCorners to detect the corners in the grayscale images. Then I drew the found corners to verify that they matched as expected.
<center>Camera Calibration</center>

![corners]

Next, I applied cv2.calibrateCamera to get the transformation matrix and the distance coefficients.
With these parameters and the cv2.undistort function, I was able to correct the distortion of the road images.
<center>Original images</center>

![original_cal]
![original]
<center>Undistorted images</center>

![undist_cal]
![undistorted]

### Color Filters
With undistorted images, I proceeded to experiment with color filters, so as to keep the white and yellow colors predominantly. These colors are the ones that are mostly used in lane lines. In order to do this, I converted the undistorted image from RGB (Red, Green, Blue) to HLS (Hue, Lightness, Saturation) colorspace. The reason for this is that the saturation channel is less sensitive to light variation than RGB. On the other hand, the lightness channel comes handy when the goal is to detect the white color. Therefore, for filtering the white lane pixels I experimented with high values of lightness thresholds with (200,255) as low and high thresholds. For filtering the yellow lanes I experimented with the hue channel to find the spectrum where the yellow color is located, together with the saturation channel to make it more insensitive to light conditions. The hue thresholds were (14,24) and the saturation thresholds were (110,255).
<center>White and yellow color filters</center>

![color_filters]

### Edge Detection
Once the lanes were filtered, they could now be more easily detected with gradient edge detection methods such as gradient direction and canny edge detection.
By experimenting with their parameters (sobel kernel size and low/high thresholds) I detected the lanes as crisp as I could get. The gradient direction edge detector (with sobel kernel size 5, and thresholds 0.8 and 1.5) had more clearly defined edges, so I kept it.

<center>Gradient direction threshold</center>

![gradient_dir]

<center>Canny edges</center>

![canny_edges]

### Perspective transforms
With the edges detected, the next step was to shift the perspective of the road to bird's view and detect the actual lines that compose the lanes. In order to do this, I first applied a polygon mask to focus the line detection to the road.
<center>Masked image</center>

![masked_image]
<center>Masked edges</center>

![masked_edges]

With the filtered image I got the perspective transform matrix with the cv2.getPerspectiveTransform function and applied the trasformation with cv2.warpPerspective. In order to get the source points for the transform, I took a straight lines example image, so I would know how the transformed image should look like (two vertical parallel lines). I plotted helper red dots in the target coordinates and calculated the offset in a way so that the number of pixels between the lines would be approximately 700. Then, I all had to do was to shift the source points so that the lines would match the red dots. The resulting coordinates are the following:

| Source   | Destination |
|----------|-------------|
| 595, 450  | 290, 0       |
| 686, 450  | 989, 0       |
| 1100, 720 | 989, 719     |
| 208, 720  | 290, 719     |

<center>Warped image</center>

![warped_image]

After the perspective transform was suitable, I applied a histogram of the lower half of the image to locate the lines' base locations. After this I used windows to capture the points in each line and slided it upwards to capture the next part of the line. If a max numberof points were captured I would recenter the window.
With all the captured points for each line, then I fitted a second order polynomial and with it I was now able to calculate the lines' curvature.
<center>Polynomial fit</center>

![polynomial]

For the case of the video lane detection, if there was previous information of a polynomial fit and its points, I would use the corresponding base points as a starting point instead of applying the histogram. Also I would use the best saved coordinate points of the previous polynomial as the windows' locations. In case not enough points were found, then I would apply the histogram at that point and recenter the following windows and slide the windows as done previously.

The last step in the pipeline was to draw the lane. I filled the space between the fitted polynomial points corresponding to each lines, and warped this figure back to the original perspective. then I could overlay it on the original undistorted image to show the detected lane. I also printed the average radius of curvature and distance from the lines to the vehicle center with the PIL library.

![lane]


### Tracking and smoothing  
As mentioned, the previous steps (a.k.a. the pipeline) were applied to a video of a car driving on a real road. They were applied frame by frame, and with help of the Line class I created, I stored information such as the following: if the lines were detected on the previous frame, the polynomial coefficients and their interpolated points, radius of curvature, and the lines' base points. For some of them, an average was kept of their last "n" observations. This was done in order to smooth the lines, as well as the reported measurements.

Video link: https://youtu.be/m6GwHx3F-nA

## 2. Discussion

### 2.1 Identify potential shortcomings with your current pipeline


I believe some shortcomings to be aware of are the following:
* In the perspective transformation, the source points are static. So if the road changes drastically (e.g. a ramp, a more pronounced curve, or simply if the car is a bit farther from the lane center), the lane lines will not be detected.
* Another car/truck occluding the lane.
* No lines whatsoever (erased due to lack of maintenance, or road covered by snow).
* Rain affecting even the HLS values.


### 2.2 Suggest possible improvements to your pipeline

I propose the following improvements:
* Use an algorithm to readjust the source points automatically
* If a car is occluding the lane, use a camera closer to the ground. If there are no lines align the car with the car ahead, or try to create a lane grid given the positions of other cars or splitting the empty road space.
* If there is snow or rain, support the camera measurements with sensors unaffected by those conditions (perhaps a laser or sonar).
