# Advanced Lane Finding
√
[//]: # (Image References)
[Chessboard]: ./output_images/chessboard_undistort.png
[TopdownView]: ./output_images/topdownview1.png
[Histogram]: ./output_images/histogram.png
[LanePixelHeuristics]: ./output_images/heuristics_for_lane.png
[SlidingWindow]: ./output_images/slidingwindowsearch.png
[LaneColorMetrics]: ./output_images/lanecolorwithmetrics.png

## Overview 
In this project, I did:
 - Camera calibration, undistorting lens effects
 - Perspective transform to project to a bird's eye view
 - Heuristics using color, gradient to identify lane pixels
 - Using a sliding window method, fitting a curve to the identified lane pixels, and marking the lane
 - Implement a tracker to use information from past frames to reduce noisy lane identifications
 - Computed curvature and distance in the real world of the camera from the center of the lane

## Camera Matrix and Distortion Coefficients
 - I used the chessboard images. Identified chessboard corners, and then `cv2.calibrateCamera()` to identify calibration matrix and distortion coefficients. 
 - Example image of chessboard with corners identified and distortion removed. There's not that much distortion in the original image.
![Chessboard][Chessboard]

## Perspective warp
 - We were given several chessboard images. I tested the matrix and coefficients I got from different images on the road test images and picked one that seemed reasonable.
 - Example road image showing original, undistorted and perspective transformed.
![Top down view][TopdownView]

 - For the co-ordinate mapping, I picked the following src and dst points:
    ```
    src = np.float32([(500, 500), (750, 500), (400, 600), (900, 600)])
    dst = np.float32([(300, 500), (750, 500), (300, 700), (750, 700)])
    ``` 
    based on what I saw from one of the road images.
 
## Heuristics to identify lane pixels
 - Applied heuristics using gradient and color values to identify lane pixels.
 - I found thresholding on min, max gradient values to be useful, and using the angle (arctan) value of the gradient to do thresholding (probably doing similar things)
 - Surprisingly, color-based thresholds were very effective. For example there were a lot of pixels from trees which were not going away with only the gradient-based thresholds. Adding a requirement on the red-channel filtered out the tree pixels. Lane pixels being yellow and white have high red channel intensities.
Example image:
![Lane pixel heuristics][LanePixelHeuristics]

## Sliding window, histogram
 - Compute histogram and use that  to identify starting position in the image for left and right lanes. 
 - And then we use a sliding window technique to gradually go up the lane. We move the window up gradually to identify nearby pixels for each of the lanes.
 - So far we have identified a set of pixels within a narrow range. We then use this to fit a second degree polynomial through `np.polyfit()` for each lane. It is important that we do this all on the warped image, as that will give us an accurate mapping of the 2d curve.
 - We then unwarp it back to the original plane using an inverse transform, and do a shading.
Example:
![Sliding Window Search][SlidingWindow]
![Histogram][Histogram]

## Curvature, distance from center of lane
We compute two metrics relatable in the physical world. Distance of camera from center of the lane. Here we just assume camera is in the center of the frame. So, we mainly compute the lane ends in phyical distance units (metres). To get mapping of pixel to metres, we use world knowledge that a typical section of the lane is 3m long, and then see how much pixels a unit distance typically covers on the image.

Curvature is computed through a formula from calculus / geometry, which just follows from the coefficients of the curve we estimated.

Example image with lane colored and metrics:
![Lane color metrics][LaneColorMetrics]

## Tracking
 - I implemented a tracker to reduce spurious lane detections (implemented as `class LaneTracker`). I keep a moving buffer for each lane. Based on the buffer history, I keep mean and standard deviation of the lane coefficients, and reject any lane curve that is too far from the mean (mean +/- 2 * sigma). When we don't have much data or sigma is 0, I use a fraction of the mean as the guide. And if there are too many rejections, then I clear my counters and start again. 
 - Without tracking, the performance was pretty good but there were a few snippets of videos where there were spurious lane detections. With tracking, that is significantly improved.
 - The final project video is [linked here](./project_video_output_tracking_compr.mp4).
 - This is a [video before tracking](./project_video_output_no_tracking.mp4)

## Discussion

This was a fun project, as I got to understand some internals of camera calibration, distortion and doing the perspective transform. I also enjoyed learning about the heuristics, sliding window and liked  building the tracker and seeing it improve the spurious detections.

### Possible Improvements
 - One thing that would have helped is some labeled data on the lanes. That would have made it easier to identify heuristics for lanes (or even better just build a classifier for lane detection from the labeled data). 
 - Having a labeled dataset would have also helped to quantify the efficacy of tracking. It could also be used to set the parameters (e.g: length of history) in a more informed way.






 

