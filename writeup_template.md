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
[image1]: ./project_images/sample_car_image.jpg
[image2]: ./project_images/sample_noncar_image.jpg
[image3]: ./project_images/sample_car_hog_image.jpg
[image4]: ./project_images/sample_noncar_hog_image.jpg
[image5]: ./test_images/test1.jpg
[image6]: ./project_images/test1_with_rects.jpg
[image7]: ./project_images/test1_heatmap.jpg
[image8]: ./project_images/test1_heatmap_with_threshold.jpg
[image9]: ./project_images/test1_with_bboxes.jpg
[image10]: ./project_images/test2_with_bboxes.jpg
[image11]: ./project_images/test4_with_bboxes.jpg
[image12]: ./project_images/test5_with_bboxes.jpg
[image13]: ./project_images/test6_with_bboxes.jpg
[video1]: ./project_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]
![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using  HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2,2)`:


![alt text][image3]
![alt text][image4]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters , color space , orientations and pixels per cell and cells per block and HOG channels. The combinations wre 
orientations  = 6,7,9,11 and 12
color space tried = RGB, HSV, LUV, HLS, YUV, YCrCb
pixels per cell = 4,8,12, and 16
cells per block =  1, 2 and 3
HOG channels = ALL or 1,2,3


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM in cell block 8

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I am doing sliding window search as part of findcars in cell  number 9. findcars function adapted from the lesson takes in the y and x start and stop position to search and the colorspace, hog channel, the trained classifier svc, scale ,
orientation, pixels per cell and cells per block

It does color transformation, sclaes the imahge based on input, (sclaes less than 1 result in some many outliers as the image is tiny to produce distinguishable HOG)

we then calculate the numnber of blocks and follow the cells to step (2 out of 8), so a overlap of 75%



#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image9]
![alt text][image10]
![alt text][image11]
![alt text][image12]
![alt text][image13]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

I am also maintaing state information of bounding boxes identified in previous video frames, I am adding the new boxed to that state. Only last 15 rectanges are kept, to refresh the state correctly as we want to throw out stale ones Now the threshold has been adjusted to 3 + half of detected rect state, to remove outliers

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here is one sample frame and their corresponding heatmaps:

![alt text][image5]
![alt text][image6]



### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image7]
![alt text][image8]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image9]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


I can make this more robust, current I am only using HOG. I can augment the features set by adding 
 binned color features, as well as histograms of color. This would result in better accuracy and robustness, i.e. with shadows. I could also have come up with a better thrshold formula to avoid outliers
 
The pipeline with likely fails if the car is making a lane change and also if this was a 2 way traffic road without a median divider (oncoming traffic will have a bigger image footprint and outlier may not be weeded out correctly)
Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

