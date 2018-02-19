**Finding Lane Lines on the Road**



### goals
* Make a pipeline that finds lane lines on the road
* Try to make the pipeline more robust to the real videos

[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. How my pipeline works

My original pipeline has the following steps:
* Convert the images to grayscale
* Do a guassian smooth, using kernel size = 3
* Run a Canny edge detection to get the gradient of the gray image
* Set region of interest and run Hough transform in this area. Draw lines based on Hough transform result 
* Combine the lines generated from Hough transform and original image. The output
the output would be the colored image with detected lanes in red.

To draw the line along the lanes, I modified the original draw_lines() function. 
* First of all, categorize the lines from Hough transform into two groups: left_lane and right_lane, by checking the slope. The lines with negative slope goes to left and that with positive slope goes to right lane. The lines with slope off too much will be filtered out. Here I am using slope of 0.3 as threshold
```
(**abs(slope) > 0.3**). 
```
* polyfit the lines in left_lane and right_lane to get an average slope(k) and interception(b) for each lane. Due to the fact that in some frames of the video, the lane conditions are so bad(e.g. no lanes marker in most area of the interest), no lines are calculated. in this case, draw_lines() will NOT draw lines. These corner cases appear to be tiny portion of the video which I think it is acceptable.
* draw_lines() is aware of the region of interest, and according to the liear equation y = k * x + b 
	* So left line could be calculated:
	x_bottomleft = int((y_bottomleft - b) / k) -> (x_bottomleft, y_bottomleft)
	x_topleft = int((y_topleft - b) / k) -> (x_topleft, y_topleft)
	* So right line could be calculated:
	x_bottomright = int((y_bottomright - b) / k) -> (x_bottomright, y_bottomright)
	x_topright = int((y_topright - b) / k) -> (x_topright, y_topright)
draw the lines for left and right lanes for each frame.
* for the challenge video, the frame size is different, thus the original region of interest is totally wrong. However a scaling down of the frame seem to work. After resizing the video from 1280x720 to 960x540, the pipeline works perfectly on it. So to make the pipeline more robust to frame, all the frames will be normalized to 960x540 before processing.
```
N_WIDTH = 960
N_HEIGHT = 540
...
img = cv2.resize(img_org, (N_WIDTH, N_HEIGHT), interpolation=cv2.INTER_LINEAR) 
```

### 2. Potential shortcomings and Possible improvement

* in my current pipeline, if the lanes suddently disappear(in most of the region of interest), draw_lines() will do nothing.
	* Maybe better to remember a "previous" line(slope/interception) assuming the road lanes change gradually rather than abruptly. Not sure if we can make this assumption in general.

* if the lanes are very curvy(e.g. some frames in challenge video) and not quite clear in large area, the slope/interception tends to have large error and the lines would deviate from lanes apprarently.
	* For the curvy road, the draw_lines() could use polynomial fit(2nd order or more) to correct the errors.


