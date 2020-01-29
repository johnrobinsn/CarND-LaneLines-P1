# **Finding Lane Lines on the Road** 
January 29, 2020

This is a write-up for Project 1 in Udacity’s Self-Driving Car Nanodegree

Project repository: https://github.com/udacity/CarND-LaneLines-P1

Project Rubric: https://review.udacity.com/#!/rubrics/322/view

    When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

    In this project you will detect lane lines in images using Python and OpenCV. OpenCV means “Open-Source Computer Vision”, which is a package that has many useful tools for analyzing images.

More about the nanodegree program: https://www.udacity.com/drive

### Reflection
My pipeline consisted of several steps that start with an RGB image taken from a dash mounted camera.  The goal is to process this raster image and arrive at determine left and right lane lines in vector-space and rendering those lane lines over the image.  An overview of the steps in my pipeline follows:

1. Convert the 3 channel rgb image of the road into a 1 channel grayscale image  for further processing.
2. Apply a gaussian blur (blur radius = 5) to smooth out noise across adjacent pixels.
3. Apply the canny edge detection algorithm to identify edges.  These edges are identified when the gradient between adjacent pixel intensities exceeds some threshold indicating the edge of an object or in this case the edge of a lane line.
4. Mask out parts of the image that are not likely to contain lane lines.
5. Run the image through the hough transform.  The hough tranform will take the raster image and will output a list of line segments that coorespond to lines (edges) found within the image.
6. Classify the lines.  First try to trivially reject lines that are not likely to be lane lines by eliminating lines that are close to horizontal.  Then try to classify the line segments as being possibly left lane lines vs right lane lines looking at both position and slope.
7. Given the line segments that are left lane candidates calculate the position and slope of the most likely left lane line.  This is done by taking the median position and slope from all of the left line samples.  We then repeat the same process for the right lane line candidates.
8. Finally draw the left and right lane lines.

Coefficients arrived at via experimentation with reviewing the pipeline.

Built the ability to look at the images at each stage in the process.

#### Optional Challenge Accepted
I did opt take up the Optional Challenge and was able to make the pipeline work effectively with the provided challenge video.  But this video did present a few additional edge cases that caused my pipeline to fail.  The first hurdle that I ran into was that when processing images that contained a yellow lane line on a lighter concrete background.

While the original pipeline worked OK with white lane lines different lighting conditions and backgrounds caused problems when analyzing yellow lines.  In order to tackle this I looked at some problematic examples.  I noticed that in many of these cases that the yellow lines were effectively lost when the image was converted to grayscale since the averaged pixel intensity of the yellow pixels were so close to the the averaged pixel intensity of the concrete backround.  But at the same time I also observed that the white lane lines worked well with the current pipeline given that while pixels are close to the maximum average pixel intensity.  Therefore I looked for a way to convert the yellow pixels in the input image into white pixels for further analysis.

#### Eliminate Yellow Pixels

Color can be represented in different coordinate systems or color spaces.  Each color space has it's own representational benefits.  The images as originally decoded are in the RGB colorspace.  The RGB colorspace used gives 8 bits of red channel data, 8 bits of green channel data and 8 bits of blue channel data.  While this color space allows you to easily manipulate the red, green or blue components of a pixel easily it's not as easy or straightward to use the RGB color space to identify which pixels are yellow.  Since yellow is a composite color.  However there are other color spaces that better separate the color component out into a single channel rather than having the color represented by 3 separate channels.  Once such color space is the HLS colorspace.  

* H - stands for Hue which represents the color of a pixel.  
* S - stands for Saturation represents the tint of the pixel.
* L - stands for Lightness respresents how lightness or darkness of the pixel.

For more information about the HLS colorspace you can refer to the [Wikipedia article](https://en.wikipedia.org/wiki/HSL_and_HSV)

The OpenCV library makes it easy to convert the image from the RGB colorspace to the HLS colorspace.  We can then define the range of hue values that represent yellow.  Given this range we can use the inRange method to generate an image mask that identifies where the yellow pixels exist within the image.  It is then a simple matter of setting the pixels to white in a copy of the original image where the mask has a non-zero value.

```python
    # First convert to the HLS colorspace so we can use the Hue component to identify yellow pixels
    hls = cv2.cvtColor(image, cv2.COLOR_RGB2HLS)
    
    # HLS ranges for selecting Yellow; values found through experimentation
    # H value (see HLS color space) opencv uses a range of 0-180 for hue
    # L value opencv uses a range of 0-255 for lightness
    # S value opencv uses a range of 0-255 for saturation
    yellow_lower = np.array([10, np.round(0.3 * 255), np.round(0.3 * 255)])
    yellow_upper = np.array([70, np.round(0.9 * 255), np.round(1.00 * 255)])
    
    # Use inRange to generate a mask that identifies all yellow pixels
    yellow_mask = cv2.inRange(hls, yellow_lower, yellow_upper)

    yellow_removed = image.copy()
    
    # Turn all of the yellow pixels identified in the yellow_mask to white
    yellow_removed[yellow_mask>0] = [255,255,255]
```

One other area of improvement was that the identify lane lines had a little more jitter than I wanted from frame to frame.  So I added some code to average the lane lines over the last ten frames when processing video clips to smooth out the lane line handling.

### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...

Additional environmental conditions
* Resiliency to mispainted lines
* Different lighting conditions (ie shadows, glare etc)
* conditions that involve rain, snow
* Other types of road surfaces
* Curvy roads

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...

Analyze more video under different environmental conditions and further tune the parameters to the various filters to handle better.  Different environmental conditions may require different parameterization and additional image processing techniques to condition the images.
