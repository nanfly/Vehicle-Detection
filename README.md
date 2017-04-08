
**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Implement a sliding-window technique and use the trained classifier to search for vehicles in images.
* Apply heatmap technique on several successive frames to filter out false positites.
* Run the pipeline on a video stream and create a new video with drawed boxes on detected vehicles.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/HOG_example_Car.jpg
[image3]: ./output_images/HOG_example_Non-car.jpg
[image4]: ./output_images/image_test.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4


### Histogram of Oriented Gradients (HOG)

#### 1. Import training data.

The code for this step is contained in the first code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here are some examples of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

#### 2. Extract HOG features from the training images.

I then explored different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]
![alt text][image3]

I chose the parameters `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)` because the result looks good and this is a pretty standard parameter set. 

#### 3. Train a SVM classifier with the extracted HOG data. 

I used a training set with 10000 images to train the SVM classifier and used a test/validation split ratio of 0.2. I firstly used `'RGB'` color space to train this model and got about 97% accuracy. Then I switched to `'YCrCb'` color space and the accuracy was improved to about 99% so I use this color space in all the following steps.

### Sliding Window Search

#### 1. Design a sliding window search pipeline to detect vehicles in a image.


I searched two scales (1.5 and 1) from y=400 to y=600 using YCrCb 3-channel HOG features. For a single image vehicle detection, I created a heatmap for the detected positives and then thresholded that map. I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. The above pipeline with threshold equals 1 seems to give good results with low false positive and high true positive. Here's an example result showing the image with multiple scale boxes, heatmap, the result of `scipy.ndimage.measurements.label()` and the final bounding boxes overlaid on the image:

![alt text][image4]
---

### Video Implementation

#### 1. Apply filter for false positives in several successive frames and generate combined bounding boxes for each frame in the video.

I apply the pipeline on the video slightly differently as on a single image. For video processing, I used a successive of frames to generate the heat map, and because I used more frames, I set the threshold higher. I played with different frame number and threshold value and found that (N = 7, t = 3) provides a decent result.  

#### 2. Here's a [link to my video result](./output_video.mp4)

---

### Discussion

My working pipleline is likely to fail when there are cars far away from the camera (meaning these car will be small in the image). Another situation my pipeline is likely to fail is when other cars are moving quickly in the video. Because my pipeline generates heatmap using bounding boxes from several successive frames, fast moving car will fail to show in the thresholded heatmap.

In general, this HOG and SVM method is not very robust becuase the accuracy is affected by the size/ orientation of the car. To gain higher accuracy using this pipeline, one need to use multiple scales of slide windows and reduce the overlap between adjacent windows. However, this will further reduce the processing speed of this algorithm, which is already very slow.  

To solve this issue, I tested another algorithm based on the YOLO netwrok, which directly predicts bounding boxes size/location/label from a image input. The result is very good and fast enough to process the video almost in real time. 

