---
title: "Tuberculosis Detector with TFLite"
date: 2019-03-08
tags: ["ml"]
description: "Building a tuberculosis detector with a modified VGG-16 model and deploying it on Android with TFLite at the PanIIT hackathon."
---

*During the PanIIT hackathon, me and my team made a tuberulosis detector trained on a slightly modified vgg-16 and we ran it on an Android phone using tflite. This is how we did it.*

## Data Sets

We got the data from [two datasets](https://ceb.nlm.nih.gov/repositories/tuberculosis-chest-x-ray-image-data-sets/) by Montgomery County and Shenzhen Hospital. This included images labeled as infected or normal and the patient age.

This is how the data was distributed initially:

![Image for initial ](/images/posts/tflite_1/image4.png)

We did a little data augumentation for the infected images and removed a few images to get the same number of images for both the types and we got the resulting distribution as:

![Image after augumentation ](/images/posts/tflite_1/image4.png)

## Data Visualization
First we had to know what the dataset looked like and how the data was distributed. So we made this graph to get the distribution for the age

![Distribution Image](/images/posts/tflite_1/image1.png)

Then we applied masking to see how if that would help us understand the data better

![Masking image](/images/posts/tflite_1/image2.png)

The masked images looked promising so we proceeded with this

## Model Selection
Now that the data was ready, our next challenge was choosing an appropriate model and then use transfer learning and train that model.
Since the dataset was so small, we couldn't train a model from scratch. We first started with MobileNet, as it is quite small and we needed to deploy the model on mobile phones, but it couldn't give us an accuracy we could work with. We tried several model after that (VGG-16, ResNet, SqueezeNet) and VGG-16 was the only one that gave an accuracy we wanted( > 80% )

![Model Selection](/images/posts/tflite_1/image7.png)

## Training the VGG-16
It was almost instantly obvious to us that we couldn't use the original model as it's size is so huge, specially the dense layers. We modified its architecture (which included adding attention to the model)

![Modified VGG-16](/images/posts/tflite_1/image8.png)

Now that we had chosen the architecture and made the necessary changes, we ran it for a couple of epochs and got an accuracy of around 89%

![Accuracy for the model](/images/posts/tflite_1/image9.png)

## Deploying the model

Now that we have our model, we had to deploy it to a mobile app. Our model was a keras model. We first converted it to an intermediate file using [this](https://github.com/amir-abdi/keras_to_tensorflow) library from github. Using this file we finally converted our model (and the weights) into a tflite compatible version (using the inbult converter)

|                     Healthy Lung                      |                     Infected Lung                      |
| :---------------------------------------------------: | :----------------------------------------------------: |
| ![Healthy Lung](/images/posts/tflite_1/image3.png) | ![Infected Lung](/images/posts/tflite_1/image6.png) |

The Android App we built was mostly structured around a sample project provided called [tensorflow for poets](https://codelabs.developers.google.com/codelabs/tensorflow-for-poets-2-tflite/)

The app performed pretty well in realtime, though we did face a lot of latency (8-9 seconds) due to the size of the model, even after the changes we made

## Final Remarks

We faced numerous problems, particularly due to tflite supporting only specific operator. In some places, we had to rewrite and retrain the model to support the inbuilt operators. In my next blog, I will show how to incorporate custom operators, as well as how to convert your models and how to run them on your device.

TFlite made us think about our model in different ways than what we were used to. There is some support for quantization, so let's see how that develops in the future

Cheers!
