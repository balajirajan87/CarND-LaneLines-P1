# **Finding Lane Lines on the Road** 

## Writeup Template

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

Below is a brief description of my pipeline. 
First, I convert the input images to gray scale. Then i equalize the histogram of intensities of the gray scale images using equalizeHist() function. This is just to make sure that there are no uneven lightinig conditions. Then i apply Gaussian Filter with Kernel size 5 just to filter out some noises and make it ready for Canny algorithm. Then i use the Canny Edge detection algorithm with low_threshold set to 200 and high_threshold set to 400. Then i apply a polygon mask to seperate out the region of interest. the vertices values are: [[(0,540),(505, 310), (505, 310), (860,540)]]. Then i use Hough transform to draw the lines based on the line shaped edges detected. The parameters are: rho=1, theta = 1rad, threshold = 15, min_line_length = 10, and max_line_gap = 15. 
I had obtained  the images with Red coloured lines drawn on both the left and the right lanes. when either one was them was discontinous, the lines was also discontinuous, and we have to join them to make it as a single line. I have also saved the output images in the test_images_output folder.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function as follows

1. within the main For Loop i first calculate the slope using: -((y2 - y1)/(x2 - x1)) (i invert the sign just to make sure that left lane line has the positive slope and the right lane line has the negative slope.
2. if the slope is above a threshold (say : 0.55, just to make sure that there are no horizontal lines visible), then it is part of left lane,and now i start to collect the y values of the lines detected ( i need to find the minimum value of that to find the top most part in a frame / image).
3. Additionally i also append the slope values to an empty array: slope_pos_array[] (just to average the same, and use the average slope at the end of the algorithm). 
4. then knowing that the y intercept of the left lane at the bottom of the image, i find the x intercept of the same at the bottom using the above calculated slope. Equation: intercept_x = x1 + ((y1 - img.shape[1])/slope). 
5. then i append the intercept to an array: left_lane_x (and i would average the same to know the average position of the lane).
6. the above said steps are repeated when the slope is negative, (i.e, for the Right Lane).
7. after the For Loop, i average the arrays, left_lane_x, and right_lane_x to get the x intercepts at the bottom for each lane.
8. Then i get the Minimum of left_lane_y and right_lane_y to get the minimum y positions of each lane to extrapolate. 
9. Then i calculate the average of slopes for both the left and right lanes from the arrays stored.
10. now with the above calculated values i get the x coordinate of the top points of the both the left and right lanes.
11. with all the above information i use: cv2.line, and x1,y1 and x2,y2 are now replaced with above calculated average infos as below

    #line with positive slope..
    cv2.line(img, (int(left_lane_x_avg), img.shape[0]), (int(left_lane_x_max), left_lane_y_min), color, thickness)
    #line with negative slope..
    cv2.line(img, (int(right_lane_x_min), right_lane_y_min), (int(right_lane_x_avg), img.shape[0]), color, thickness)

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


1. One Potential Short comming is : i use a fixed slope threshold: 0.55 for both left and righ lanes.. This is kept assuming that camera position and caliberation is fixed.. now when the Camera is moved to another position then, this would not work. 
2. another potential shortcomming is this wont work when the road does not have a lane marking. ???


### 3. Suggest possible improvements to your pipeline

Two possible improvements:
1. the input to canny can be improved a bit, (to improve the detection of Lane lines). Maybe we can capitalize on HSV Colour spaces, and or calculate the texture of the image near the Lane segments and Extract the lane lines alone with Image segmentation techniques. 
2. create curved lines that fit to the Curved lanes. 
