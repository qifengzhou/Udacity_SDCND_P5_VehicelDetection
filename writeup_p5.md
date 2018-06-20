---
output:
  pdf_document: default
  html_document: default
---
## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/Scan_road.jpg
[image2]: ./output_images/Scan_road_combined.jpg
[image3]: ./output_images/Scan_road_heatmap.jpg
[image4]: ./output_images/Scan_road_pipeline.jpg
[image5]: ./output_images/Car_NotCar.jpg
[image6]: ./output_images/Car_NotCar_HOG.jpg

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 'get_hog_features' function starting from line #7 in 'function_utils.py'.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![Car NotCar][image5]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YUV` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![Car NotCar HOG][image6]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, it seems that the most imortant one is 'pix _per_cell', using 16 and 32 will get a image without good representation of cars and not cars. And 8 will produce a reasonable impression to distinguish the car and not car classes. Having 9 or 11 for 'orient' is not so different. I use 9 for computation. 'cell_per_block' is working better with 2 than 4.  

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using
'color_space = 'YCrCb' # Can be RGB, HSV, LUV, HLS, YUV, YCrCb
orient = 9  # HOG orientations
pix_per_cell = 8 # HOG pixels per cell
cell_per_block = 2 # HOG cells per block
hog_channel = o # Can be 0, 1, 2, or "ALL"
spatial_size = (16, 16) # Spatial binning dimensions
hist_bins = 16    # Number of histogram bins
spatial_feat = True # Spatial features on or off
hist_feat = True # Histogram features on or off
hog_feat = True # HOG features on or off
y_start_stop = [400, 700] # Min and max in y to search in slide_window()'

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):
'y_buffer = [[400, 490], [410,500], [400,500],[420,520], [400,560],[430,590], [400,650],[410,710]]
xy_windows = [(30,30), (30,30), (50,50), (50, 50), (80,80), (80,80), (150,150), (150,150)]'

The road to the horizon is searched with smaller windows, and gradually to the nearer scene with bigger windows. They are overlapped to better cover the cars with light colors. 

![Slide windows][image1]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on 3 scales using YCrCb o-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![Windows combined][image2]
![Windows heatmap][image3]
![Frame pipeline][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
The video produced frame by frame is 'test_video_output.mp4'

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I took last -1, -3, -5, -7, -9 frames of detection to overlap with current frame to get one heatmap to reduce positives. The code is in the code cell 'video_process()' function. 

And the last 22 frame are archived to keep the video pipeline detection smooth.
The code is in 'add_detect()' in 'Vehicel_Detection()'.

The previous frames are stored in the 'Vehicle_Detection()' class. 

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This is a hardcoded car detection pipeline. The paramters were searched manually. It may fail in many situation, like the cars it didn't see before, false positives in sequential frames.. 

A deep learning aproach like YOLO would work better, though higher computation power is needed. When accuracy is more favored than computation power, it is worth a try. And this system can complement with the deep learning approach to have a higher confidence of the detections.  

