# Self-Driving Car Engineer Nanodegree Program,
## Robust Lane Finding Project using advanced Computer Vision Techniques

<img src=\"output_images/data_drawn.png\" width=\"480\" alt=\"Combined Image" />

The goals / steps of this project are the following:
* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.\n",
* Apply a distortion correction to raw images."
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image (\"birds-eye view\")
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)
    [im01]: ./output_images/camera_calibration.png "Chessboard Calibration"
    [im02]: ./output_images/distortion_correct1.png "Undistorted Chessboard"
    [im03]: ./output_images/distortion_correct2.png "Undistorted Dashcam Image"
    [im04]: ./output_images/unwarped.png "Perspective Transform"
    [im05]: ./output_images/unwarp_hsv_channel.png "Colorspace Exploration"
    [im06]: ./output_images/sobel_magnitude_direction.png "Sobel Magnitude & Direction"
    [im07]: ./output_images/hls_l_channel.png "HLS L-Channel"
    [im08]: ./output_images/hls_b_channel.png "LAB B-Channel"
    [im09]: ./output_images/pipeline_all_test_images.png "Processing Pipeline for All Test Images"
    [im10]: ./output_images/sliding_windows_polyfit.png "Sliding Window Polyfit"
    [im11]: ./output_images/sliding_windows_histogram.png "Sliding Window Histogram"
    [im12]: ./output_images/polyfit_from_prev_fit.png "Polyfit Using Previous Fit"
    [im13]: ./output_images/lane_drawn.png "Lane Drawn onto Original Image"
    [im14]: ./output_images/data_drawn.png "Data Drawn onto Original Image"
    [im15]: ./output_images/unwarp_lab_channel.png "Colorspace Exploration"
    [im16]: ./output_images/unwarp_rgb_channel.png "Colorspace Exploration"
    [im17]: ./output_images/hls_s_channel.png "HLS S-Channel"
    [im18]: ./output_images/sobel_absolute.png "Sobel Absolute"
    [im19]: ./output_images/sobel_direction.png "Sobel Direction"
    [video1]: ./project_video_output.mp4 "Video"
    [video2]: ./challenge_video_output.mp4 "Video"
    [video3]: ./harder_challenge_video_output.mp4 "Video"


### Writeup / README

    #### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.
    
    ### Camera Calibration
    #### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image. The OpenCV functions `findChessboardCorners` and `calibrateCamera` are used for the image calibration. 3D points -object points and 2D image points - image points are calculated using these functions. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. Arrays of object points, corresponding to the location (essentially indices) of internal corners of a chessboard, and image points, the pixel locations of the internal chessboard corners determined by `findChessboardCorners`, are fed to `calibrateCamera`. `calibrateCamera` returns the camera matrix, distortion coefficients, rotation and translation vectors etc. These coefficients are constant for a given camera (and lens). These can then be used by the OpenCV `undistort` function to undo the effects of distortion on any image produced by the same camera. The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function `drawChessboardCorners`
    ![alt text][im01]

    The image below depicts the results of applying `undistort`, using the calibration and distortion coefficients, to one of the chessboard images:

    ![alt text][im02]

    ### Pipeline (single images)
    #### 1. Provide an example of a distortion-corrected image.
    The image below depicts the results of applying `undistort` to one of the project dashcam images:
    ![alt text][im03]
  
    #### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
    
    Next is to use the Sobel Gradients and Color channel thresholds. I explored several combinations of sobel gradient thresholds and color channel thresholds in multiple color spaces.
    
    We need to pass a single color channel to the `cv2.Sobel()` function, so first convert it to grayscale - `cv2.cvtColor(im, cv2.COLOR_RGB2GRAY)`

    Below is an example of absolute sobel threshold:
    ![alt text][im18],
    Below is an example of the combination of sobel magnitude:
    ![alt text][im19]

    Below is an example of the combination of sobel magnitude and direction thresholds:
    ![alt text][im06]
    The below image shows the various channels of three different color spaces for the same image:
    ![alt text][im05]
    ![alt text][im15]
    ![alt text][im16]
    
    Below are examples of thresholds in the HLS L channel, HLS S channel and the LAB B channel:
    ![alt text][im07]
    ![alt text][im08]
    ![alt text][im17]

    I chose to use just the L channel of the HLS color space to isolate white lines and the B channel of the LAB colorspace to isolate yellow lines.
    Below are the results of applying the __complete pipeline__ to various sample images:
    ![alt text][im09]
    
    #### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
    The function `unwarp` implements the perspective transform. A perspective transform maps the points in a given image to different, desired, image points with a new perspective. The perspective transform you’ll be most interested in is a bird’s-eye view transform that let’s us view a lane from above; this will be useful for calculating the lane curvature later on. A perspective transform can also be used for all kinds of different view points.
    Steps include:
    * Compute the perspective transform, M, given source and destination points `cv2.getPerspectiveTransform(src, dst)
    * Compute the inverse perspective transform (Minv) `cv2.getPerspectiveTransform(dst, src)
    * Warp an image using the perspective transform, M: `cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_LINEAR)`
    * The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points instead of doing it programmatically. Many perspective transform algorithms will programmatically detect four source points in an image based on edge or corner detection and analyzing attributes like color and surrounding pixels. The image below demonstrates the results of the perspective transform:
    ![alt text][im04]
    #### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
    The functions `sliding_window_polyfit` and `polyfit_using_prev_fit`, which identify lane lines and fit a second order polynomial to both right and left lane lines. The first of these computes a histogram of the bottom half of the image and finds the bottom-most x position (or \"base\") of the left and right lane lines. Originally these locations were identified from the local maxima of the left and right halves of the histogram, but in my final implementation I changed these to quarters of the histogram just left and right of the midpoint. This helped to reject lines from adjacent lanes. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively \"follows\" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy `polyfit()` method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:
    ![alt text][im10]
    The image below depicts the histogram generated by `sliding_window_polyfit`; the resulting base points for the left and right lanes - the two peaks nearest the center - are clearly visible:
    ![alt text][im11]
    The `polyfit_using_prev_fit` function performs basically the same task, but alleviates much difficulty of the search process by leveraging a previous fit (from a previous video frame, for example) and only searching for lane pixels within a certain range of that fit. The green shaded area shows where we searched for the lines this time. So, once you know where the lines are in one frame of video, you can do a highly targeted search for them in the next frame. This is equivalent to using a customized region of interest for each frame of video, and should help you track the lanes through sharp curves and tricky conditions. If you lose track of the lines, go back to your sliding windows search or other method to rediscover them. The image below demonstrates this - the green shaded area is the range from the previous fit, and the yellow lines and red and blue pixels are from the current image:
    ![alt text][im12]
    
    #### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
    The radius of curvature is based upon [this website](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) and calculated in the code cell titled \"Radius of Curvature and Distance from Lane Center Calculation\" using this line of code (altered for clarity)
     `curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])`
    In this example, `fit[0]` is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and `fit[1]` is the second (y) coefficient. `y_0` is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). `y_meters_per_pixel` is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.
    The position of the vehicle with respect to the center of the lane is calculated with the following lines of code: `lane_center_position = (r_fit_x_int + l_fit_x_int) /2
     center_dist = (car_position - lane_center_position) * x_meters_per_pix`
    `r_fit_x_int` and `l_fit_x_int` are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).
    Function `calc_curv_rad_and_center_dist()` calculates the radius of curvature based on real world space (from pixel space). For this project, I assumed that if you're projecting a section of lane similar to the images above, the lane is about 30 meters long and 3.7 meters wide. So here's a way to repeat the calculation of radius of curvature after correcting for scale in x and y as mentioned in the notes -
    * Define conversions in x and y from pixels space to meters\n",
    `ym_per_pix = 30/720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 # meters per pixel in x dimension`
    * Fit new polynomials to x,y in world space
    `left_fit_cr = np.polyfit(ploty*ym_per_pix, leftx*xm_per_pix, 2),
     right_fit_cr = np.polyfit(ploty*ym_per_pix, rightx*xm_per_pix, 2)`
    * Calculate the new radii of curvature,
    `left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])`,
    * Now our radius of curvature is in meters
    __Radius of curvature: 454.166802555 m, 1274.44473829 m
    Distance from lane center: -0.398372165719 m __
   
   #### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
   
    I implemented this step in the code cells titled \"Draw the Detected Lane Back onto the Original Image\" and \"Draw Curvature Radius and Distance from Center Data onto the Original Image\" in the Jupyter notebook. A polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix `Minv` and overlaid onto the original image. The image below is an example of the results of the `draw_lane` function:\n",
   ![alt text][im13]
    Below is an example of the results of the `draw_data` function, which writes text identifying the curvature radius and vehicle position data onto the original image:
    ![alt text][im14]
   
   ### Pipeline (video)\n",
    #### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
    Here are the links to various videos generated using the pipeline above [Project video]
    project_video_output.mp4)
    [Challenge video](./challenge_video_output.mp4)
    [Harder challenge video](./harder_challenge_video_output.mp4)
    
### Discussion
    #### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
    The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc. It wasn't difficult to dial in threshold parameters to get the pipeline to perform well on the original project video (particularly after discovering the B channel of the LAB colorspace, which isolates the yellow lines very well), even on the lighter-gray bridge sections that comprised the most difficult sections of the video. It was trying to extend the same pipeline to the challenge video that presented the greatest challenge. The lane lines don't necessarily occupy the same pixel value (speaking of the L channel of the HLS color space) range on this video that they occupy on the first video, so the normalization/scaling technique helped here quite a bit, although it also tended to create problems (large noisy areas activated in the binary image) when the white lines didn't contrast with the rest of the image enough.  This would definitely be an issue in snow or in a situation where, for example, a bright white car were driving among dull white lane lines.
    Below are some more considerations to make it more robust
    a) Different types of curve fits
    Euler spirals, are parametric curves whose curvature changes linearly with the independent variable. They are frequently used in highway engineering to design connecting roads, on and off ramps etc. These might be a better candidate curve to fit to the lane pixels rather than simple second order polynomials.
    b) Gradient enhancement
    A color image can be converted to grayscale in many ways. The easiest is what is called equal-weight conversion where the red, green and blue values are given equal weights and averaged. However, the ratio between these weights can be modified to enhance the gradient of the edges of interest (such as lanes). According to Ref.2, grayscale images converted using the ratios 0.5 for red, 0.4 for green, and 0.1 for blue, are better at detecting yellow and white lanes. This method was used for converting images to gray scale in this project.
    
    ### References
    [1] Juneja, M., & Sandhu, P. S. (2009). Performance evaluation of edge detection techniques for images in spatial domain. International Journal of Computer Theory and Engineering, 1(5), 614.
    [2] Yoo, H., Yang, U., & Sohn, K. (2013). Gradient-enhancing conversion for illumination-robust lane detection. IEEE Transactions on Intelligent Transportation Systems, 14(3), 1083-1094.
    [3] McCall, J. C., & Trivedi, M. M. (2006). Video-based lane estimation and tracking for driver assistance: survey, system, and evaluation. IEEE transactions on intelligent transportation systems, 7(1), 20-37.
