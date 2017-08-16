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


**The rubric followed for this project can be found: [Here](https://review.udacity.com/#!/rubrics/571/view)**

---

**Camera Calibration**

The code for this step is the functions ```readinCalibrate()``` and ```undistort(img, objpoints, imgpoints)``` located in the third code cell of the IPython notebook located: [Here](https://github.com/DavidG1011/Udacity-Advanced-Lane-Lines--P4/blob/master/Project.ipynb).

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


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
