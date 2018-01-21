# **Finding Lane Lines on the Road**

---

The purpose of this project is to develop software to find lane lines on road images, as coming from a camera in a car.
The software is developed using a set of fixed images, and afterwards tested on 3 different video streams.

The following techniques are used:
* Color and area selection
* Canny edge detection
* Hough transformation line detection


[//]: # (Image References)

[originalimage]: ./test_images/solidWhiteCurve.jpg "Example of development image"
[hlsimage]: ./test_images_output/hls/Image0.jpg "HLS color transformation"
[whiteyellowimage]: ./test_images_output/white_yellow/Image0.jpg "White and yellow colors selected"
[croppedimage]: ./test_images_output/cropped/Image0.jpg "Area of interest selected"
[cannyimage]: ./test_images_output/canny/Image0.jpg "Canny edges"
[houghimage]: ./test_images_output/hough/Image0.jpg "Hough lines on orginal image"
[finalimage]: ./test_images_output/final/Image0.jpg "Averaged lanes lines on orginal image"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps, in the following order:

* Selection of relevant colors
* Selection of relevant region
* Canny edge detection
* Hough line transformation
* Calculate left and right lane lines

**Step 1: Selection of relevant colors**

The code was developed using a set of 6 fixed images. Here is an example:

![alt text][originalimage]

In order to select the relevant white and yellow colors I converted the image to the HLS color space. This color space is better when detecting colors in case of shades or other problematic light situation. The HLS image looks like this:

![alt text][hlsimage]

I defined ranges for the values of the H, L and S components and masked the unwanted colors out. The result are images like this:

![alt text][whiteyellowimage]

**Step 2: Selection of relevant region**

Next step in my pipeline is to select the relevant region of the white/yellow images. I defined a polygon in the lower half of the image and masked everything outside this away. The result looks like this:

![alt text][croppedimage]

I chose to perform the area selection at this point in the pipeline, because the following steps (Canny edge detection and Hough Line Transformation) are processing intensive, so reducing the work to be done seems reasonable.

**Step 3: Canny edge detection**

Nest step is to perform Canny edge detection. Edges are detected by finding where the image changes color. In order to do this effectively images are first converted to gray scale, and second blurring is applied to smoothe rough edges. The result of Canny edge detection looks like this:

![alt text][cannyimage]

**Step 4: Hough line transformation**

The next step in the pipeline is to apply Hough line transformation to find straight lines. This technique utilises that a straight line in image space becomes a single point in hough (rho, theta) space. If several pixel points sit on the same straight line on the (x,y) image of edges, they will map to the same point in (rho, theta) space. So finding straight lines becomes a matter of finding pixels who map to the same (rho, theta) value.

The result of Hough line transformation looks like this:

![alt text][houghimage]

**Step 5: Calculate left and right lane lines**

The result of step is a number of different lines. In order to display them properly average left and right lane lines need to be calculated.
The Hough lines are represented by the end points (x1,y1) and (x2,y2), and that is not very good for averaging. So I convert the line end points (x1,y1) and (x2,y2) into the form of a line slope S and an intersection with the y-axis y0. S and y0 can be averaged in a meaningful way. Further I calculate the length L of the individual hough line segments and use those as weighting factors when calculating the averages of S and y0. It makes sense to give more weight to longer line segments when averaging.  

The resulting image looks like this:

![alt text][finalimage]

I did not actually modify the draw_lines() function directly, but wrote my own functions for drawing the single lane lines. My average_lines() function calculates the average lines as described above, and returns a slope S and y-intersection y0 for the left and right lane lines. My draw_lane_lines() function essentially turns the S,y0 value into 2 points (x1,y1) and (x2,y2) located in the lower 40% of the images and draws the lines via the OpenCV function addWeighted on top of the original image.

### 2. Identify potential shortcomings with your current pipeline

One shortcoming of this pipeline is that is works only for approximately straight lines. It can be seen on the challenge video that the detected lane lines become a little "nervous" when the road is curved. With nervous I mean that they flicker a bit back and forth sometimes. I reckon we will learn how to handle curves later in the nanodegree.

Another potential shortcoming is the performance in darkness and shades. I have tried to accomodate for this by using the HLS color transformation. It remains to be proven how well this actually works as the test videos were pretty friendly in terms of light.

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to average the detected lines over a number of images in the video. This might remove the flickering which I described as nervousness above.

Obviously the handling of curves need to be added for this pipeline to be useful.
