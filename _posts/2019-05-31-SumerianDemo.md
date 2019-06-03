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

# Chatbot with Lex

# Webcam

# Recognition
