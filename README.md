## Writeup Template



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

[image1]: ./writeup_images/undistorted_calibration_image.png "Undistorted"
[image2]: ./test_images/straight_lines1.jpg "Original Image"
[image3]: ./output_images/undistorted_test_images/straight_lines1.jpg "Undistorted Image"
[image4]: ./output_images/color_gradient_images/SobelX_images/straight_lines1.jpg "SobelX"
[image5]: ./output_images/color_gradient_images/SBinary_images/straight_lines1.jpg "SBinary"
[image6]: ./output_images/color_gradient_images/LBinary_images/straight_lines1.jpg "LBinary"
[image7]: ./output_images/color_gradient_images/RBinary_images/straight_lines1.jpg  "RBinary"
[image8]: ./output_images/color_gradient_images/straight_lines1.jpg "Binary Output"
[image9]: ./writeup_images/PerspectiveTransform.png "PT"
[image10]: ./output_images/warped_images/straight_lines1.jpg "Warped binary"
[image11]: ./output_images/polyfit_images/1.png "Polyfit line"
[image12]: ./output_images/final_result_images/straight_lines1.jpg "Final Result"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
Please find the the intermediate (step by step)output images for the test images [here](https://github.com/AkshathaHolla91/CarND-Advanced-Lane-Lines/tree/master/output_images)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

##### Original Image
![alt text][image2]

Using 'cv2.undistort()'  function as mentioned above, the test image has been undistorted as shown below to reduce the effects of distortion on the image which were leading to problems like change in the shape, size and distance of objects and lane lines in the image as compared to the original scene. These problems have been eliminated after undistortion

##### Undistorted Image
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell 7 of the jupyter notebook in  `advanced_lane_lines.ipynb`). I have combined SobelX gradient , Red channel of RGB colourspace ,Saturation and Lightness of HLS colorspace for performing the transformation by taking the individual binary values and then combining and stacking them.

The SobelX gradient allows us to identify the vertical edges in an image and hence helps us to identify most portions of the lane lines. I have used a sobel kernel of 7 over the default value of 3 to obtain a smoother gradient and given thresholds of (10, 190) to obtain a decent result as seen below.

##### Sobel X gradient binary image
![alt text][image4]

When each individual channel is identified in the HLS colorspace , the Saturation channel is able to produce the lane lines clearly even in varying light conditions (brightly lit roads) and also able to detect the high saturation values of the yellow and white lines well , hence I have used it with a threshold range of (150,255 ) to include necessary values, similarly skipping the lower values in the Lightness channel helps us to counter the effects of shadows in the image and helps us to identify lane lines better. So I have used the L channel with thresholds of (120, 255) to skip the lower lightness values ie shadows which may affect detection of lines.

##### S Binary image
![alt text][image5]

##### L Binary image
![alt text][image6]

When identifying individual channels in RBG colorspace for the images, I found that the Red channel at higher thresholds help us to identify the lane lines better than the other 2 channels and when combined with Sobel X gradient and Saturation channels gives an image with lane lines identified clearly with minimal noise. Hence I have used R channel with the threshold range (200,255)

##### R binary image
![alt text][image7]

All the images shown above are binary images of individual channels and gradients. I also tried combinations of Sobel Y , Magnitude and Gradient direction apart from the channels above, but felt that the R, L ,S and SobelX combination worked better for me.

The end result obtained by combining the binaries is shown below.


##### Color Gradient transorm binary result
![alt text][image8]




#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspectiveTransform()`, which appears in the cell 8 of the jupyter notebook.  The  function takes as inputs an image (`image`). The source and destination points that I chose for the calculation of the transformation matrix are shown below.

| Source        | Destination   | 
|:-------------:|:-------------:| 
|230, 693     | 350,700       | 
| 592,450      | 350, 0     |
| 687,450     | 900,0      |
| 1076,693      | 900,700        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
An example of the same is shown below.

![alt text][image9]

 Please note that I have used a region of interest mask on the color gradient binary output before giving it as an input to the perspective transform to eliminate noise due to other objects surrounding the lane. The resultant warped binary is shown below.

![alt text][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I have used the sliding window approach for identifying the lane line pixels as seen in cell 10 of the python notebook. Here the histogram of the lower half of the warped image is taken to identify the areas with highest pixel intensities on the left half and the right half of the image to identify the base of the left and right lane lines. Further windows are drawn from the base of each line with a height of around 80pixels each and a margin of 100 pixels on each side to identify the non zero ie lane pixels in each window and further the mean of the lane pixels identified  is used for the calculation of the next window offset from the current window and so on.The identified lane pixels help to draw  lines by using a second order polynomial function ie polyfit function which gives us an approximate of the lane lines.  In this way the lane pixels are identified in each window in each  line and lines connecting the corresponding lane pixels are drawn and identified. 

The following image shows the lane pixels for the left line in red
,the right line in blue , the identified windows in green and the fitted line/ curve in yellow.

![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I have calculated the radius of curvature of the lane lines in cell 12 of the notebook using the formula R_curve=(1+(2Ay+B)^2)^(3/2)/ |2A|. Here we calculate the radius of curvature at the bottom of the image ie base of the lane lines and the co-efficients A and B are obtained from the left curve polyfit and the right curve polyfit. This is later converted from pixel space to world space ie pixels to meters by multiplying with the following values
ym_per_pix = 30/720 ( meters per pixel in y dimension)
xm_per_pix = 3.7/700(meters per pixel in x dimension)
Finally the average of the radius of curvature of the 2 lines is considered.

The position of the vehicle is calculated by identifying the center of the car which is assumed to be the center of the image and subtracting it from the center of the lane which is calculated by finding the midpoint between the 2 lane lines as seen in cell 13


 
​	

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 14 of the notebook  in the function `plotLane()`.  Here is an example of my result on a test image:

![alt text][image12]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://github.com/AkshathaHolla91/CarND-Advanced-Lane-Lines/blob/master/final_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

While implementing the pipeline steps as seen above, the main challenge faced was tuning the color transforms and the Gradient transform combinations to obtain an optimal combination so as to identify the lane lines under different lighting conditions and curvatures.Initially I started with the sobelX gradient for detecting the vertical lines and tried combining it with sobel y and Magnitude and gradient direction binaries. The resulting binary output gave me the lane lines but did not perform entirely well on the video since the areas with light and shadows were not handled properly and also due to the presence of noise the lane lines were fluctuating a lot. Then I tried combining Saturation channel binary and still found it unsatisfactory. This led me to explore the effects of individual channels in the HLS and RGB colorspaces and I found that the combination of R channel with S channel and SobelX gradient worked well. Further I also added L channel with high threshholds for reducing the effect of shadows. I have also used Region of Interest mask as in the first project to ensure reduction of noise due to presence of unnecessary elements.

Even after applying the above methods there is still slight fluctuation in the lane lines when the lighting conditions change.I hope to try and improve this further by experimenting with more color channels and gradient transforms in the future.

Potential improvements to the pipeline could be finding a generic way to define source and destination points for the perspective transform and also a pipeline which was not restricted to a particular lane length and width.

Another improvement would be to design a pipeline which would factor in obstacles in the lane which is a highly possible scenario.

The pipeline implemented above can also be improved by ensuring continuity in lane detection even when there are no lane lines in the current frame by considering the average of the lane line pixels in the previous few frames.

