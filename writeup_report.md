**Advanced Lane Finding Project**

[//]: # (Image References)

[chessboard]: ./camera_cal/calibration4.jpg 
[0_original_image]: ./output_images/0_original_image.png 
[1_undistorted_image]: ./output_images/1_undistorted_image.png 
[2_thresholded_image]: ./output_images/2_thresholded_image.png 
[3_perspective_transformed_image]: ./output_images/3_perspective_transformed_image.png 
[4_final_image]: ./output_images/4_final_image.png 
[camera_process]: ./examples/camera_process.png
[video1]: ./project_video_processed.mp4 "Video"


Overall my pipeline has the following stages:
1. Camera Calibration
2. Un-distort Image
3. Binary Thresholding
4. Perspective Transform
5. Locate Lanes
6. Fit Lanes
7. Draw Lanes

Let's dive into each step in detail

## 1. Camera Calibration

In this stage, series of chessboard images are read from camera_cal directory. 

![Chessboard Image][chessboard]

cv2.findChessBoardCorners() is used to find the x,y coordinates of each corner on a given image.
WIth the array of corners, cv2.calibrateCamera() is used to calibrate camera matrix, distortion coefficients vector &
the camera vectors of rotation & translation.


## 2. Un-distort Image

After the camera is calibrated, we can apply camera matrix & distortion coefficients to correct the distortion effects 
on camera input images. This is done using cv2.undistort().

Original Image:
![Original Image][0_original_image]

Un-distorted Image
![Un-distorted Image][1_undistorted_image]


## 3. Binary Thresholding

Thresholding stage processes the undistorted image, masks out pixels that are part of lane & removes that are not.

Following is the mini pipeline used:

![Camera Process][camera_process]

The input RGB image is converted into Grayscale, then apply a sobel filter in the X direction to get image edges that match 
the direction of the lane lines. Then to filter out pixels that are of not interest a threshold function is applied.
Min / Max of 30 & 150 threshold is used to produce a binary image of pixel of interest.

Similarly, the other path converts input RGB image into HLS color space. We use S-Channel of that. Again to filter out pixels
it's passed through InRange function. Min / Max of 175 & 250 seems to work good to produce a binary image of pixel of interest.

By combing above two, Threshold Binary & InRange Binary, Combined binary is generated as below:

![Combined Image][2_thresholded_image]

## 4. Perspective Transform

This transforms the image into a bird's eye view. cv2.wrapPerspective() is used for this.   
Preset coordinate list is used for perspective transform, cv2.getPerspectiveTransform() generates a perspective
matrix.

![Perspective Transformed image][3_perspective_transformed_image]

## 5. Locate Lanes

This stage extracts the actual lane pixels of left & right lanes from the above the image. 

For the cases where lanes were not identified, we generate a histogram of the bottom half of the image.
Use the two peaks of the histogram, a good starting point for image pixels at the bottom of the image are
identified called as x_left & x_right.

Now the image is divided into 10 horizontal strips of equal size. All the noise is discarded starting from the bottom strip.
After doing this for all strips, we are left with pixels of left & right lanes.

## 6. Fit Lanes

With the above image, Numpy's polyfit() is used to fit a polynomial equation to each lane.
Then with the Radius of Curvature caluclated validate if it's good enough as per US government specifications for highway
curvature.

The second check is to see that left_x & right_x coordinates where the intersect the bottom of the image have not changed
too much from frame to frame.

The third check is to radius of curvature is within 100x the previous frame values.

If any of above fail, then it's considered that lane is lost & values from previous frame are used.

## 7. Draw Lanes

Finally, the detected lanes are drawn back on the undistored image. 
The final output looks like this:

![Final Image][4_final_image]

