# **Finding Lane Lines on the Road** 
January 29, 2020 - John Robinson

This is a write-up for **Project 1** in Udacity’s Self-Driving Car Nanodegree

My Project repository: https://github.com/johnrobinsn/CarND-LaneLines-P1

Project Rubric: https://review.udacity.com/#!/rubrics/322/view

    When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

    In this project you will detect lane lines in images using Python and OpenCV. OpenCV means “Open-Source Computer Vision”, which is a package that has many useful tools for analyzing images.

More about the nanodegree program: https://www.udacity.com/drive


 <img src="examples/laneLines_thirdPass.jpg" width="380" alt="Combined Image" />


### Reflection
To sucessfully complete this project, I needed to develop an image processing pipeline that would detect left and right lane lines from a static image.  

My pipeline consisted of several steps that start with an RGB image taken from a dash mounted camera.  The goal is to process this raster image and derive the likely left and right lane lines in vector-space and render those lane lines over the image.  An overview of the steps in my pipeline follows:

1. Convert the 3 channel RGB image of the road into a 1 channel grayscale image for further processing.
2. Apply a gaussian blur (blur radius = 5) to smooth out noise across adjacent pixels.
3. Apply the canny edge detection algorithm to identify edges.  These edges are identified when the gradient between adjacent pixel intensities exceeds some threshold indicating the edge of an object or in this case the edge of a lane line.
4. Using a polygon in the shape of a isoceles trapezoid, I masked out parts of the image that are not likely to contain lane lines. 
5. We then run the resulting image through the hough transform.  The hough tranform will take the raster image and will output a list of line segments that coorespond to lines (edges) found within the image.  It is in this stage that we transition out of processing a raster image and into processing lines in vector space.
6. Classify the lines.  First try to trivially reject lines that are not likely to be lane lines by eliminating lines that are close to horizontal.  Then try to classify the line segments as being possible left lane lines vs right lane lines by looking at both position and slope.
7. Given the line segments that are left lane candidates calculate the position and slope of the most likely left lane line.  This is done by taking the median position and slope from all of the left line samples.  We then repeat the same process for the right lane line candidates.
8. Finally draw the left and right lane lines onto the original image.

Coefficients for the canny and hough transform were arrived at via experimentation to look for values that worked reasonably well given the test images and videos.  In order to review the effects of these coefficiants, I added a mechanism to easily look at the image processing at various stages of the pipeline.

#### Optional Challenge Accepted
I did take up the Optional Challenge and was able to make the pipeline work effectively with the provided challenge video.  This video presented a few additional edge cases that caused my pipeline to fail.  The first hurdle was when my pipeline failed to detect yellow lane lines on a lighter concrete background.  In order to tackle this, I analyzed some problematic examples.  I noticed that in many of the failcases the yellow lines were effectively lost when the image was converted to grayscale since the averaged pixel intensity of the yellow pixels were so close to the the averaged pixel intensity of the lighter concrete backround.  At the same time, I also observed that the white lane lines worked well with the current pipeline given that white pixels are close to the maximum possible pixel intensity.  Therefore I looked for a way to convert the yellow pixels in the input image into white pixels as an enhancement to the pipeline image processing.

#### Eliminate Yellow Pixels
Color can be represented in different coordinate systems or color spaces.  Each color space has it's own representational benefits.  The images as originally decoded are in the RGB colorspace.  The RGB colorspace used gives 8 bits of red channel data, 8 bits of green channel data and 8 bits of blue channel data.  While this color space allows you to easily manipulate the red, green or blue components of a pixel, it's not as easy or straightward to use the RGB color space to identify which pixels are yellow given that it is a composite color.  However there are other color spaces that better separate the color component out into a single channel rather than having the color represented by 3 separate channels.  One such color space is the HLS colorspace.  

* H - Hue which represents the color of a pixel.
* L - Lightness respresents how lightness or darkness of the pixel.  
* S - Saturation represents the tint of the pixel.

For more information about the HLS colorspace you can refer to the [Wikipedia article](https://en.wikipedia.org/wiki/HSL_and_HSV)

The OpenCV library makes it easy to convert the image from the RGB colorspace to the HLS colorspace.  We can then define the range of hue values that represent yellow.  Given this range we can use the inRange method to generate an image mask that identifies where the yellow pixels exist within the image.  It is then a simple matter to use the mask to change the yellow pixels to white.  Below is a small python/opencv code snippet that transforms yellow pixels into white pixels.

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

    # We'll change the yellow pixels to white pixels on a copy
    # of the original image
    yellow_removed = image.copy()
    
    # Turn all of the yellow pixels identified in the yellow_mask to white
    yellow_removed[yellow_mask>0] = [255,255,255]
```

At this stage the pipeline worked well even on the challenge video but the lane lines jittered a little more than I wanted from frame to frame.  So I added some code to average the lane lines over the last ten video frames to smooth out the lane line handling.

### 2. Identify potential shortcomings with your current pipeline
Although I'm quite impressed with how well this simple image processing pipeline handles the provided example images and videos, I'm sure that other challenging scenarios would expose additional weakpoints.  Some additional environmental conditions that I think would provide additional edge cases follow:

* Mispainted lines
* Different lighting conditions (ie shadows, glare etc)
* Weather conditions that involve rain, snow
* Other types of road surfaces
* Curvy roads
* Larger gaps betwen lane lines.

### 3. Suggest possible improvements to your pipeline
Different environmental conditions may require different parameterization and additional image processing techniques to condition the images.  This would likely require some ability to dynamically detect differing conditions and have slightly different pipelines for these conditions. 

## Running the Project
This project is in the form of a jupyter notebook hosted in the following github repository:

The easiest way to run this notebook for yourself with all of the required dependencies is to use docker.  If you have docker installed simply run the following command:

```
docker run -it --rm --entrypoint "/run.sh" -p 8888:8888 -v `pwd`:/src udacity/carnd-term1-starter-kit
```

Given this command docker will download the specified docker image and launch a jupyter notebook server with all of the right environmental bits inside the container.  You can then open a browser pointed at http://localhost:8888.  You will have to be copy and paste the token that is printed out to the console by the jupyter notebook server.  Once in jupyter, navigate to P1.ipynb and open it.


