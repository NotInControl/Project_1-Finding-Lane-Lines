# **Finding Lane Lines on the Road** 

## Kevin Harrilal Project Submission

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following steps. 

First, I converted the images to HSV format, then I created two separate tresholding images. One to identify areas where the pixel had a white color, and other to identify areas of pixels that had a yellow color. This was done using HSV where the Hue for white in OpenCV is 0,180 because that is where the circle rolls over, and the Hue for yellow is 30. A range around similar hue, saturation, and value values for both yellow and white were used to allow for better robutsness. 

After the two tresholding images were created, they were combined into a single image by using the bitwise OR operation. 

The point of the above was to reduce as much non-essential information as posisble before trying to detect edges, and since we can assume the lane markings are yellow or white for this exercise, that assumption was applied to the image.

The output of the canny filter is a set of pixels that represents the edges of the images. 

I then use the HoughLines function to transform the the image from image space to hough space. This is helpful because lines can easily be detected in hough space. Where a line in image space may be represented by mx+b, in hough space, it is represented by a point. So 

Once a single thresholded images is had, identifying areas of yellow and white, I pass the image to a the canny edge detector which identidy areas where the gradient is between 50 and 150. The values were determined by trial and error and represent how different the values are in adjacent pixels.

The way to find lines in image space is to look for intersecting lines in hough space. Where many lines in hough space intersect, we can infer that we have detected a line in image space. This is done automatically with the HoughLines function, where the output is the set of contiguos lines, but to do so the function takes in a set of parameters: rho, theta, threshold, which control the angular resultion of the grid space used by the hough function specified in radians, threshold which identifies how many intersections a particular grid needs before it will be deteremined as a line, min_line_length which represents the minimum lenght of a line in pixels, and max_line_gap, which is the maximum distance in pixels between segments that are considered the same line. 

Each of these paramaters were manually adjusted in an interative process until the output was satisfactory, as shown in the plots of the python notebook. 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function with an algorithm that calculates all of the slopes from each line segment returned by the hough line function. The assumption is that similar sloping lines in similar regions of the picture are in fact a part of the same line, and so a single line should be drawn on the final image.
    
To have a better estimate for the line, the average slope of all similar lines for the left and right sides (right should have negative slopes, left should have positive slopes), are averaged, for a single line to be drawn. Slopes and intercepts that are within 15% of eachother are considered to be on the same line, and then the average is taken to draw the final line. 

The image size and the point of the beginning line is used to draw a line from the bottom of the image to a point approximately 57% high into the image. The same process is repeated for both lanes. 

For the videos in particular, it was noticed that the pipeline would sometimes not detect a line for the dashed lane lines, the solid lines (white or yellow) never showed any issues. 

To help combat this issue, an additional treshold was added to the video pipeline. 1st a grayscalled image was used, and then bitwised or'ed with the threshold result of the white, and yellow image. The purpose of this was to have more pronounced pixels before going to the egde detector. 

Since the lane line function uses slope, we can add more detail to the image, and allow the slope algorithm determine which slopes to filter out vs keep for the same line. The effect is having more information to determine similar lines by adding the grayscaled image into the pipeline, and the issue was no longer seen



### 2. Identify potential shortcomings with your current pipeline


There are a few shortcomings with the approach. 1st off, while the output was done on many different types of images and videos, a lot of the parameters were manually tuned, and since we are still using color space tresholding, I do not think this solution will scale well for many different lighting/shadowing conditions. 

This is actually seen in some of the videos where the lane marking falls under a shadow, they are not detected as well. 

Another issue with this approch has to do with the fact that all lane markings are modeled by a single line. In reality this is not true, and while the output looks great on the videos with straight lines, on curved roads the single line model does not fit the marking very well. In some instances the offset, or slope are averaged so drastically that the lane line is drawn askew of the actual marking in the video. Overall it is a great predictor, but I think there is still room for improvement. 


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to a higher order polynominal to fit the points detected. Like drawing a quintic spline through discreet nodes on the line. 

Another possible improvement is to run the image though many more thresholding filters, to account for changes in yellow vs white markings in day vs night, or have a variable (vs) fixed thresholding setup like we do to account for different operating points, similar to gain scheduling on PID controllers. 
