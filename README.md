# **Finding Lane Lines on the Road** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

# Overview - original task description

When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project you will detect lane lines in images using Python and OpenCV.  OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.  

To complete the project, two files will be submitted: a file containing project code and a file containing a brief write up explaining your solution. We have included template files to be used both for the [code](https://github.com/udacity/CarND-LaneLines-P1/blob/master/P1.ipynb) and the [writeup](https://github.com/udacity/CarND-LaneLines-P1/blob/master/writeup_template.md).The code file is called P1.ipynb and the writeup template is writeup_template.md 

To meet specifications in the project, take a look at the requirements in the [project rubric](https://review.udacity.com/#!/rubrics/322/view)


# Result

I created a traditional CV-based solution that works reasonably well on the test videos, 
but obviously could not scale to more complex scenarios. 

- Description of the pipeline can be found in [writeup_template.md](writeup_template.md).
- The actual implementation is in [P1.ipynb](P1.ipynb#Actual-implementation---test-single-images).
- Open [P1.html](P1.html#Actual-implementation---test-single-images) if you can not render
the notebook.

## Sample video 

the right separator is a bit hard to see, sorry about that
![Sample output video](test_videos_output/solidWhiteRight.gif)

## Main steps of the pipeline

![Main steps of the pipeline](steps_plot.png)
