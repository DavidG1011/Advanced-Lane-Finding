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


[image1]: /output_images/distorted.png "Undistorted"
[image2]: /output_images/undistortedimage.png "Undistorted Pic"
[image3]: /output_images/colorandsobel.png "Binary Process"
[image4]: /output_images/linesdrawn.png "Source Drawn"
[image5]: /output_images/diagramoftransf.png "Transform Diagram"
[image6]: /output_images/binarytransformed.png "Binary Transformed"
[image7]: /output_images/histogram.png "Histogram"
[image8]: /output_images/coloredlines.png "Colored Lines"
[image9]: /output_images/overlaydata.png "Overlayed"

---

**The rubric followed for this project can be found: [Here](https://review.udacity.com/#!/rubrics/571/view)**

**[Notebook](https://github.com/DavidG1011/Udacity-Advanced-Lane-Lines--P4/blob/master/Untitled.ipynb)**

---

**Camera Calibration**

The code for this step is the functions ```readinCalibrate()``` and ```undistort(img, objpoints, imgpoints)``` located in the third code cell of the IPython notebook located: [Here](https://github.com/DavidG1011/Udacity-Advanced-Lane-Lines--P4/blob/master/Untitled.ipynb).

The former function reads in a collection of checkerboard images taken from various angles and with various distortions. The images are converted to grayscale and then the cv2 function ```cv2.findChessboardCorners()``` is used to find the corners of the checkerboard. A corner being an edge where 2 white and 2 black squares intersect. These corners are then used to create image points which will be mapped to undistorted object points. These points are then used in the cv2 function ```cv2.calibrateCamera()``` to generate the camera matrix and distortion coefficients. These are finally used in the cv2 function ```cv2.undistort()``` to generate an image that is free of distortion.

A visual example of this undistortion is shown below:

![alt text][image1]


---

**Pipeline for [Project](https://github.com/DavidG1011/Udacity-Advanced-Lane-Lines--P4/blob/master/Project.ipynb) Notebook**

---

**1. Undistortion:**

The first step of the pipeline is to undistort the incoming images. This is to ensure that each image is properly represented, which is especially important when doing something as visual as lane line detection. Once each checkerboard image is read in once with ```readinCalibrate()```, it is no longer necessary to do, as we have the correct image points to apply to other images for undistortion. Therefore, only ```undistort()``` is used for each new image. Again, these functions are located in code cell 3 of the above notebook. 

![alt text][image2]

---

**2. Color Spaces, Binary Transformations:**

For this project, I decided to use a combination of HSV color space thresholding and an x direction sobel operation.

First, I used my function ```convertHSV()``` located in code cell 3 to convert passed in images to HSV color space by using the cv2 function ```cv2.cvtColor(img, cv2.COLOR_RGB2HSV)``` then,  I created a binary image of the same dimensions as the input image and thresholded my desired color values for each channel of HSV. I wanted to be able to detect both white and yellow lane colors in my images, and set my thresholds as such:

**HSV Values:**

| Color      | Low Thresh    |       High Thresh|  
|:-------------:|:----------:|:----------------:| 
| White    | 20,0,180    |      255,80,255  | 
| Yellow     |  10,100,90 |    22,220,255 |


For the sobel operator, I used my function ```sobelx()``` --also located in code cell 3, to grayscale an image and apply the ```cv2.Sobel()``` function. Similarly to the HSV binary, I also created one for the sobel operator and applied a threshold of 30 for the low and 100 for the high. I also applied blur with a kernel size of 7 to smooth unwanted pixel data.

Finally, I blended the 3 binary images together by creating a blank image and drawing found pixels in each individual frame onto the complete frame. The code for this process can be found in code cell 6 of the notebook.

A visual representation of this process:


![alt text][image3]


---

**Perspective Transformation:***

The code used to transform images can be found in code cell 3 of the python notebook, particularly the function ```transform()```. This function uses source and destination points to convert the passed in image into an overhead perspective. This is accomplished with hardcoded source and destination points. The points chosen for this transformation are as follows:

| Source      | Destination   | 
|:-----------:|:-------------:| 
| 529,466     | 139,0         | 
| 751,466     | 1141,0        |
| 1218,675    | 1141,720      |
| 62,675      | 139,720       |

These points are put into an array and used with the function ```cv2.getPerspectiveTransform(src, dst)``` to caculate the perspective transform. ```cv2.warpPerspective()``` is then used to perform the perspective transformation. 

Some visuals to demonstrate:


**Source Points Drawn:**


![alt text][image4]


Points drawn to mark out the area of interest. 


**Transformation Applied** 


![alt text][image5]


A diagram of the process. The ```tranform()``` function from code cell 3 also has an option to invert the transformation by simply swapping the source and destination order in the ```cv2.getPerspectiveTransform()``` function. This inverse transform is labeled above.
The code to perform this process on a single frame is located in code cell 7 of the python notebook.



This process can then also be applied to the binary images demonstrated in the previous section.


**Binary Transformed:**


![alt text][image6]


This binary transform code can be seen for a single frame in code cell 8


---

**Detecting Lane Pixels**

The code for the process of identifying lane pixels is contained in a function called ```drawTransformLines()``` in code cell 3 of the python notebook. This code is referenced from Project: Adavnced Lane Lines, Pt 33. Finding Lane Lines

The first step of the process is to create a histogram of the lower half of the transformed binary image. This is a good starting point because you can separate the histogram down the middle and somewhat confidently identify the strongest concentration of pixels on the left half of the histogram as the left lane, and the strongest right side as the right lane.

![alt text][image6]


By creating a histogram of the bottom half of the above image, we can identify lane lines a good portion of the time. The code to display the histogram of a single frame is located in code cell 9. 

![alt text][image7]


As can be seen above, the left lane is clearly identified. However, in this particular image, the right lane is not entirely confidently identified. To counter this, I moved the midpoint of the histogram over about 200 pixels so that the center between the lane lines is not accidentally identified as a lane line in the case of excess image noise from shadows, debris, etc. This works because the lane lines are always going to be a certain distance apart and, theoretically, the right lane will never be that close to the left lane.


Next, I continued the steps of the sliding window approach. This approach entails dividing the source image into slices and identifying non-zero pixels in each slice, then concatenating those indices to be used with ```np.polyfit()``` to fit a second order polynomial to each index. These detected lines are then colored red and blue to represent the respective lane. A line is also calculated and fit over to have a visual representation of where each lane line is. This line calculation and drawing can be seen for a single frame in code cell 10 of the python notebook. 


![alt text][image8]


---

**Calculating Radius And Offset:**

The code for these 2 calculations is in code cell 3.

Using function ```curveAndOffset()```. This function takes the fitted lines detailed above and calculates the radius of the lines in pixels, using ```ym_per_pix = 30./720``` and ```xm_per_pix = 3.7/700``` as a rough conversion of meters to pixel space. 

The offset is calculated by finding the lane center: ```lane_center = (left_fitx[-1] + right_fitx[-1])/2```, and calculating how far the lane center is from the center of the image ```lane_center = (left_fitx[-1] + right_fitx[-1])/2```. This output is then converted to pixel space by using the conversion above: ```xm_per_pix = 3.7/700```. 

These are then overlayed on the output image for easy visual representation.


![alt text][image9]


The code for a single frame of this can be seen in code cell 11 of the python notebook. Note: This is actually the full pipeline for video processing only used on one frame, so the overlayed lane marker is visible.


---

**Plotting The Lane Marker**


The code for drawing the lane marking is in code cell 3, function ```drawUntransformLines()``` 


by using ```cv2.fillPoly()``` and our old friend ```transform()``` in inverse, we can draw a polygon over our detected lane lines on the perspective transformed binary and then do an inverse perspective transform to have a natural space marking of where our detected lines are. This is then combined with our original colored and, of course, undistorted image using ```cv2.addWeighted()``` This gives us our final overlayed output, as also shown above:


![alt text][image9]





---


**Final Pipeline** 

The final video pipeline for this project is located in, you guessed it, code cell 3. function: ```videopipeline()```.

Code for a single frame of the pipeline can be run from cell 11 in the python notebook. Code for the full pipeline is in code cell 12.



Here's a [link to my video result](/Final_Ouput.mp4)


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
