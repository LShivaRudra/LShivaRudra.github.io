---
title: Drone Detection Using DL Methods
summary: Implemented drone detection using Yolo algorithms and techniques like layer freezing, image augmentation. Performed comparitive study of various scenarios and test cases
tags:
  - Deep Learning
  - Python
date: '2023-01-09T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

image:
  caption: 
  focal_point: Smart

links:
  - icon: github
    icon_pack: fab
    name: GitHub Link
    url: https://github.com/LShivaRudra/CvBasedDroneDetection/
url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
# slides: example
---
## **Introduction**
Unmanned Aerial Vehicles (UAVs), commonly known as drones have witnessed a massive increase in the past few years. Drones are not only being used for recreational purposes but also in a vast number of applications in engineering, disaster management, logistics, securing airports, etc. Extensive usage of drones has immediately raised security concerns due to the potential of their use in malicious activities. To address this problem, this project is aimed at analyzing the available drone detection solutions which can identify drones from the day and night camera feeds in real-time. This study also aims to identify the potential challenges involved in drone detection and study the various accuracy and loss metrics in drone detection problems. Finally, the drone detection solutions will be compared and a suitable COTS drone detection solution will be suggested such that it gives the best trade-off between false alarm and miss. 
This project is focused on studying the nature of the YOLOv3 algorithm in object detection and understanding, implementing the methods used for improving the working of the algorithm in terms of giving better outputs for the given test data. 

For the implementation of drone detection, state of the art YOLO algorithm has been utilized.

## **Methodology**
### Dataset Preparation:
- Preparation of the input dataset for YOLO requires two folders: Images and their corresponding Labels(consists of ‘.txt’ files). The name of the files in the Images folder should match the corresponding label file names in the Labels folder. 
- First, we obtained a dataset with around 1800 images and their labels of drones from ‘Kaggle’. We divided the dataset into ‘train’ and ‘val’ folders that stand for ‘training’ and ‘validation’. 
- The training set is used to train the model and to form the corresponding weights and biases that will be applied whenever we test our model with any new image. The validation set is used to validate/verify the results obtained after training the data.
- But, there are limited drone images available open-source. Also, to improve the performance, it is required to add ‘dummy data’ like images of flying birds, planes and blank images of sky. Along with these, we even included images with a combination of drones and birds. This allows the model to properly distinguish between both of them.
- It is also advisable to use variations of images like blurred, greyscale, images with noise, random cropping, flipping, rotating of images, color jittering etc. This is called ‘Data Augmentation’. It helps in improving accuracy and preventing overfitting.

### Data Augmentation:
- In image classification, it is usually recommended  to randomly distort the spatial characteristics, for eg. randomly flip, rotate and crop images in order to improve accuracy and avoid overfitting.
- In object detection, one has to perform augmentation with more caution as object detection algorithms can be quite sensitive to such changes.
- Random geometry transformation includes random cropping,  random horizontal flip, random expansion and random resize.
- Random color jittering includes changing brightness, hue, saturation, and contrast.
- For single-stage detectors, such as YOLO, random geometry transformation is encouraged.

## **Results**
### YoloV3 on Dataset without Bird Dataset or Augmentation:
Epoch count: 30
Batch Size: 16
Results:



![1.png](/images/DroneDetectionYolo/1.png)



![2.png](/images/DroneDetectionYolo/2.png)

The model however incorrectly detected birds as drones as shown below. This could be a potential hassle during surveillance. We tried to resolve this by including birds in our dataset and adding it as a new label.


![3.png](/images/DroneDetectionYolo/3.png)

The paper had used a dataset of over 10000 images and ran over 150 epochs. Our systems were unable to sustain the load and it exceeded the limits of the free version of google colab.

### Training YoloV3 with Data Augmentation:
![4.png](/images/DroneDetectionYolo/4.png)
The mAP has significantly improved compared to the earlier case.


A lot of augmentation was done on the bird dataset because the number of training examples in it was significantly lesser than the drones.



Results after 10 epochs:

![5.png](/images/DroneDetectionYolo/5.png)

Results are pretty good with an improved mAP.


The following is the detection of a bird and a drone with the improvements applied:


![6.png](/images/DroneDetectionYolo/6.png)

## **Experimentation**
### Training on original drone dataset using yolov3 for 50 epochs:
![7.png](/images/DroneDetectionYolo/7.png)
Different Augmentation Techniques like
- Random Cropping
- Brightness Change
- Cutout
- Mosaic 
have been done

Ran on 25 epochs without the bird dataset:

![8.png](/images/DroneDetectionYolo/8.png)
It can be seen that the augmented dataset performs better than the original dataset but it tends to overfit after about 5 epochs. While the original dataset had an mAP@0.5 of 0.1592, the augmented dataset had an mAP@0.5 of 0.632 which is a significant improvement. 
The tendency of the augmented dataset to overfit can be combated by performing Early Stopping. Hence, we eventually ran it for 10 epochs.

### Training YoloV5 by Freezing Layers:
Yolov5 uses CSPDarknet53 as a backbone. This backbone integrates gradient change into the feature map, resolves the repeating gradient information in wide backbones, which improves accuracy, minimizes the inference speed, and decreases the model size by reducing the parameters.Experimental results reveal that YOLOv3 outperforms YOLOv5 in terms of speed. However, YOLOv5 had the best recognition accuracy. The following are the architecture differences.
- In object detection algorithms, we generally use models which have been trained on the MS COCO dataset and fine tune it for our own dataset and we train all layers of this model.
- Pretrained models are powerful and hence we need not train the entire model. We can skip training a few layers of the model by using the ‘--freeze’ argument while executing the train.py script.
- When we freeze a few layers, those layers don't get trained and hence backpropagation does not happen through those layers which reduces the training time.
- Even though we don’t train a few layers, though the mAP of the frozen layer model is lower than the normal model, the difference is nominal and can hence be ignored.

### Results for Layer Freezing:

We train the model using three different pre-trained weights: Yolo 5s(small), YoloV5m(medium) and YoloV5m with frozen layers. The S version has 7.2 million parameters while the 5 version has 21.2 m parameters.

![9.png](/images/DroneDetectionYolo/9.png)



![10.png](/images/DroneDetectionYolo/10.png)
