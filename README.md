
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/cover.png" alt="alt text" align="middle"/>

<p align="center"><i>built with deep convolutional neural network and ❤️</i></p>

## Table of Contents
1. [Motivation](#1-motivation)
2. [The Database](#2-the-database)
3. [The Model](#3-the-model)
	* [3.1 Input Layer](#31-input-layer)
	* [3.2 Convolutional Layers](#32-convolutional-layers)  
	* [3.3 Dense Layers](#33-dense-layers)
	* [3.4 Output Layer](#34-output-layer)
	* [3.5 Deep Learning](#35-deep-learning) (in progress)
4. [Model Validation](#4-model-validation) (in progress)
	* [4.1 Performance](#41-performance)
	* [4.2 Analysis](#42-analysis)
	* [4.3 Computer Vision](#43-computer-vision)
5. [Application Requirements](#5-application-requirements) (in progress)
	* [5.1 RESTful API](#51-restful-api)
	* [5.2 Interactive Web App](#52-interactive-web-app)
	* [5.3 Real-Time Prediction via Webcam](#53-real-time-prediction-via-webcam)
6. [Next Steps](#6-next-steps)  (in progress)
7. [About the Author](#7-about-the-author)
8. [References](#8-references)

## 1 Motivation
Human facial expressions can be easily classified into 7 basic emotions: happy, sad, surprise, fear, anger, disgust, and neutral. Our facial emotions are expressed through activation of specific sets of facial muscles. These sometimes subtle, yet complex, signals in an expression often contain an abundant amount of information about our state of mind. Through facial emotion recognition, we are able to measure the effects that content and services have on the audience/users through an easy and low-cost procedure. For example, retailers may use these metrics to evaluate **customer interest**. Healthcare providers can provide better service by using additional information about **patients' emotional state** during treatment. Entertainment producers can monitor **audience engagement** in events to consistently create desired content.

> **“2016 is the year when machines learn to grasp human emotions”**
--Andrew Moore, the dean of computer science at Carnegie Mellon.

Humans are well-trained in reading the emotions of others, in fact, at just 14 months old, babies can already tell the difference between happy and sad. **But can computers do a better job than us in accessing emotional states?** To answer the question, I designed a deep learning neural network that gives machines the ability to make inferences about our emotional states. In other words, I give them eyes to see what we can see.

## 2 The Database
The dataset I used for training the model is from a Kaggle Facial Expression Recognition Challenge a few years back (FER2013). It comprises a total of **35887 pre-cropped, 48-by-48-pixel grayscale images** of faces each labeled with one of the 7 emotion classes: anger, disgust, fear, happiness, sadness, surprise, and neutral.

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/fer2013.png" width="500" align="middle"/>
<h4 align="center">Figure 1. An overview of FER2013.</h4>
</p>

As I was exploring the dataset, I discovered an imbalance of the “disgust” class (only 113 samples) compared to many samples of other classes. I decided to merge disgust into anger given that they both represent similar sentiment. To prevent testing set leakage, I built a data generator [fer2013datagen.py](https://github.com/JostineHo/mememoji/blob/master/src/fer2013datagen.py) that can easily separate training and hold-out set to different files. I used 28709 labeled faces as the training set and held out the remaining two test sets (3589/set) for after-training validation. The resulting is a 6-class, balanced dataset, shown in Figure 2, that contains angry, fear, happy, sad, surprise, and neutral. Now we’re ready to train.

<img src="https://github.com/JostineHo/mememoji/blob/master/figures/trainval_distribution.png" alt="alt text" align="middle"/>
<h4 align="center">Figure 2. Training and validation data distribution.</h4>

## 3 The Model
<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/mrbean.png" width="200" align="middle"/>
<h4 align="center"> Figure 3. Mr. Bean, the model for the model.</h4>
</p>

Deep learning is a popular technique used in computer vision. I chose convolutional neural network (CNN) layers as building blocks to create my model architecture. CNNs are known to imitate how the human brain works when analyzing visuals. I will use a picture of Mr. Bean as an example to explain how images are fed into the model, because who doesn’t love Mr. Bean?

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/netarch.png" width="650" align="middle"/>
<h4 align="center">Figure 5. Facial Emotion Recognition CNN Architecture (modification from Eindhoven University of Technology-PARsE).</h4>
</p>

A typical architecture of a convolutional neural network will contain an input layer, some convolutional layers, some dense layers (aka. fully-connected layers), and an output layer.  These are linearly stacked layers ordered in sequence. In [Keras](https://keras.io/models/sequential/), the model is created as `Sequential()` and more layers are added to build architecture.

###3.1 Input Layer
* This layer has pre-determined, fixed dimensions, so the image must be pre-processed before it can be fed into the layer. I used [OpenCV](http://docs.opencv.org/3.1.0/d7/d8b/tutorial_py_face_detection.html#gsc.tab=0), a computer vision library, for face detection in the image. The `haar-cascade_frontalface_default.xml` in OpenCV contains pre-trained filters and uses `Adaboost` to quickly find and crop the face.
* The cropped face is then converted into grayscale with `cv2.cvtColor` and resized to 48-by-48 pixels with `cv2.resize`. This step greatly reduces the dimensions compared to the original RGB format with three color dimensions (3, 48, 48).  The pipeline ensures every image can be fed into the input layer as a (1, 48, 48) numpy array.

###3.2 Convolutional Layers
* The numpy array gets passed into the `Convolution2D` layer where I specify the number of filters as one of the hyperparameters. The **set of filters** (aka. kernel) are non-repetitive with randomly generated weights. Each filter, (3, 3) receptive field, slides across the original image with shared weights to create a new feature map.
*  The convolution step generates **feature maps** that represent the unique ways pixel values are enhanced, for example, edge and pattern detection. In Figure 4, a feature map is created by applying filter 1 across the entire image. Other filters are applied one after another creating a set of feature maps.

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/conv_maxpool.png" width="600" align="middle"/>
<h4 align="center">Figure 4. Convolution and 1st max-pooling used in the network</h4>
</p>

* **Pooling** is a dimension reduction technique usually applied after one or several convolutional layers. It is an important step when building CNNs as adding more convolutional layers can greatly affect computation time. I used a popular method called `MaxPooling2D` that uses (2, 2) windows each time only to keep the maximum pixel value. As seen in Figure 4, max-pooling on the (2, 2) square sections across the feature map results in a dimension reduction by 4.

###3.3 Dense Layers
* The dense layer (aka fully connected layers), is inspired by the way neurons transmit signals through the brain. It takes a large number of input features and transform features through layers connected with trainable weights.
* The network can be initialized by applying random weights to the interconnected layers. These weights can then be trained using **back propagation** with labeled data. Back propagation looks at errors produced in the output and back calculates the weight corrections to every layer before. The weights are gradually adjusted as we feed in more training data allowing the model to learn patterns.
#####MATH
* By tuning the hyper-parameters, such as **learning rate** and **network density**, we can control the training speed and the complexity of the network architecture.
* Essentially, the more layers/density we increase in the model the better it can pick up signals. As good as it may sound, the model also becomes increasingly prone to overfitting the training data. One method to prevent overfitting and generalize on unseen data is to apply **dropout**. Dropout randomly selects a portion (usually less than 50%) of nodes to set their weights to zero during training. This method can effectively control the model's sensitivity to noise in the training data while maintaining the necessary complexity of the architecture.
#####MATH
* It is important to note that there is no specific formula to building a neural network that would guarantee to work well. Different problems would require different network architecture and a lot of trail and errors to produce desirable validation accuracy. This is the reason why nets are often perceived as "black box algorithms." But don't be discouraged. Time is not wasted when experimenting to find the best model and you will gain valuable experience.  

###3.4 Output Layer
* Instead of using sigmoid activation function, I used **softmax** at the output layer. This output presents itself as a probability for each emotion class.
#####MATH
Therefore, the model is able to show the detail probability composition of the emotions in the face. As later on, you will see that it is not efficient to classify human facial expression as only a single emotion. Our expressions are usually much complex and contain a mix of emotions that could be used to accurately describe a particular expression.

###3.5 Deep Learning
To begin with, I built a simple CNN with an input, three convolution layers, one dense layer, and an output layer. As it turned out, the simple model preformed poorly. The low accuracy of 0.1500 showed that it was merely random guessing an emotion. The CNN architecture was too simple to pick up the subtle details in facial expressions. This could only mean one thing...

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/inception.png" width="500" align="middle"/>
</p>

This is where **deep learning** comes in. Given the pattern complexity of facial expressions, it is necessary to build with a deeper architecture in order to identify subtle signals. So I tested various net architectures that goes from complex to more complex. I played around with three components: number of convolutional layers, portions of dropout, and number of dense layers. In the end, my final net architecture was 9 layers deep in convolution with one max-pooling after every three convolution layers as seen in figure below.

+ Talk about [automate.py]() and GPU computing on AWS
+ Analysis of various arch
+ Pros/Cons
+ Comparison Tables

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/netarch.png" width="300" align="middle"/>
<h4 align="center">Figure 5. Facial Emotion Recognition CNN Architecture.</h4>
</p>


## 4 Model Validation

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/works_every_time.png" width="500" align="middle"/>
</p>
###4.1 Performance
As it turns out, the final CNN had a **validation accuracy of 58%**. This makes a lot of sense, because it is hard to use one label to classify each facial expression. Our expressions usually consist a combination of emotions. When the model predicts incorrectly on one label, the correct label is often the second most likely emotion as in examples in Figure 8 with blue labels.

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/predictions.png" width="700" align="middle"/>
<h4 align="center">Figure 8. Random 24 examples in test set validation.</h4>
</p>

###4.2 Analysis
<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/true_pred.png" width="300" align="middle"/>
<h4 align="center">Figure 9. True emotion label and model prediction count on test set.</h4>
</p>

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/confusion_matrix.png" width="400" align="middle"/>
<h4 align="center">Figure 10. Confusion matrix for true and prediction emotion counts.</h4>
</p>

###4.3 Computer Vision
As a result, the feature maps become increasingly abstract down the pipeline when more pooling layers are added. Figure 6 and 7 gives an idea of what the machine sees in feature maps after 2nd and 3rd max-pooling.

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/conv64pool2.png" width="400"align="middle"/>
<h4 align="center">Figure 6. CNN (64-filter) feature maps after 2nd layer of max-pooling.</h4>
</p>

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/conv128pool3.png" width="600" align="middle"/>
<h4 align="center">Figure 7. CNN (128-filter) feature maps after 3nd layer of max-pooling.</h4>
</p>

## 5 Application Requirements

<p align="center">
<img src="https://github.com/JostineHo/mememoji/blob/master/figures/system.png" width="400" align="middle"/>
<h4 align="center">Figure 11. Application pipeline.</h4>
</p>

I built 3 applications around this project.

###5.1 RESTful API
 + built on AWS EC2 instance
 + with mongoDB feedback storage
 + Find source code here.
 + Find the API [RESTful API](http://54.227.229.33:5000/static/swagger-ui-build/index.html) Try it!
 + requirements...

###5.2 Interactive Web App
- Find the app: [MemeMoji --the app](http://54.227.229.33:5000/static/FaceX/index.html) Try it!
- Find source code here [link]
- special thanks
- requirements...

###5.3 Real-Time Prediction via Webcam
- Real-time emotion prediction OpenCV
- Find source code here [link]
- requirements...

## 6 Next Steps

+ maxout
+ collective emotion score

(to be continued...)

## 7 About the Author

**Jostine Ho** is a data scientist who loves building intelligent applications and exploring the exciting possibilities using deep learning. She is interested in computer vision and automation that creates innovative solutions to real-world problems. She holds a masters degree in Petroleum & Geosystems Engineering at The University of Texas at Austin. You can reach her on [LinkedIn](https://www.linkedin.com/in/jostinefho).

## 8 References

1. [*"Dataset: Facial Emotion Recognition (FER2013)"*](https://www.kaggle.com/c/challenges-in-representation-learning-facial-expression-recognition-challenge/data) ICML 2013 Workshop in Challenges in Representation Learning, June 21 in Atlanta, GA.

2. [*"Andrej Karpathy's Convolutional Neural Networks (CNNs / ConvNets)"*](http://cs231n.github.io/convolutional-networks/) Convolutional Neural Networks for Visual Recognition (CS231n), Stanford University.

3. Srivastava et al., 2014. *"Dropout: A Simple Way to Prevent Neural Networks from Overfitting"*, Journal of Machine Learning Research, 15:1929-1958.

4. Duncan, D., Shine, G., English, C., 2016. [*"Report: Facial Emotion Recognition in Real-time"*](http://cs231n.stanford.edu/reports2016/022_Report.pdf) Convolutional Neural Networks for Visual Recognition (CS231n), Stanford University.
