---
layout: post
title: Sumerian concierge with facial recognition
---
This tutorial will show you how to create a basic virtual host on AWS Sumerian, able to talk with you, recognize you, detect your emotion.

# Introduction

![_config.yml]({{ site.baseurl }}/images/Demo_scene2.png)

### Features

You can talk with the Sumerian host by pressing the Space bar or the Microphone button. 

Ask the host activate the webcam in order to detect your emotions and recognize your face.

### Prerequisites

- Webcam, Microphone and Speakers
- Sign in to Amazon Sumerian with your AWS account

### Technologies used

- [Amazon Sumerian](https://aws.amazon.com/sumerian/) :  Used to create scene, handle user interaction and display a virtual host with which you can interact as a chat bot
- [Amazon Lex](https://aws.amazon.com/lex/) : Service for building conversational interfaces using voice and text as a chat bot. This service is used to create intents for the Sumerian host and to react to the users requests.
- [Amazon Rekognition](https://aws.amazon.com/rekognition/) :  Image and video recognition service allowing facial recognition, emotion analysis.
- [Amazon DynamoDB](https://aws.amazon.com/lex/) : AWS database that will be used to save face ID and user's names.
- [Tracking.js](https://trackingjs.com/) : OpenCV based Javascript library allowing to detect faces on videos and images.

![_config.yml]({{ site.baseurl }}/images/tech.png)

# Create the scene

First of all, you will need to set up your Sumerian scene by granting all AWS access you need in it. Therefore, you will need to create a Cognito Identity Pool by following [this tutorial](https://docs.sumerian.amazonaws.com/tutorials/create/beginner/aws-setup/). The Cognito identity Pool will provide Sumerian Users a temporary token allowing you to use a AWS services from Sumerian such as Lex, Rekognition...

![_config.yml]({{ site.baseurl }}/images/cognito.png)

Once the Cognito Identity Pool is created by selecting the Polly and Lex template, you can get the Identity Pool (red box) and configure your Sumerian scene with it.

By default, you will only be able to access Lex and Polly. To add the Rekognition and DynamoDB access, click on the role link (red box), and add attach policies to it as bellow.

![_config.yml]({{ site.baseurl }}/images/role.png)



# Chatbot with Lex

# Webcam

# Recognition
