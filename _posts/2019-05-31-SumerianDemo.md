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
- Sign in to Amazon Sumerian with your [AWS account](https://signin.aws.amazon.com/signin?client_id=signup&redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&page=resolve)
- Download the [default scene asset](../download/sumerianhostrecognition-bundle.zip)
- Download [files](../download/filesToS3.zip) to put on S3
- Be on point on [scripting](https://docs.sumerian.amazonaws.com/tutorials/create/beginner/scripting-basics/index.html) and [state machine](https://docs.aws.amazon.com/sumerian/latest/userguide/sumerian-statemachines.html) 

### Technologies used

- [Amazon Sumerian](https://aws.amazon.com/sumerian/) :  Used to create scene, handle user interaction and display a virtual host with which you can interact as a chat bot
- [Amazon Lex](https://aws.amazon.com/lex/) : Service for building conversational interfaces using voice and text as a chat bot. This service is used to create intents for the Sumerian host and to react to the users requests.
- [Amazon Rekognition](https://aws.amazon.com/rekognition/) :  Image and video recognition service allowing facial recognition, emotion analysis.
- [Amazon DynamoDB](https://aws.amazon.com/lex/) : AWS database that will be used to save face ID and user's names.
- [Tracking.js](https://trackingjs.com/) : OpenCV based Javascript library allowing to detect faces on videos and images.

![_config.yml]({{ site.baseurl }}/images/tech.png)

# Init the scene

### Grant AWS access

First of all, you will need to set up your Sumerian scene by granting all AWS access you need in it. Therefore, you will need to create a Cognito Identity Pool by following [this tutorial](https://docs.sumerian.amazonaws.com/tutorials/create/beginner/aws-setup/). The Cognito identity Pool will provide Sumerian Users a temporary token allowing you to use a AWS services from Sumerian such as Lex, Rekognition...

![_config.yml]({{ site.baseurl }}/images/cognito.png)

Once the Cognito Identity Pool is created by selecting the Polly and Lex template, you can get the Identity Pool ID (red box) and configure your Sumerian scene with it.

By default, you will only be able to access Lex and Polly. To add the Rekognition and DynamoDB access, click on the role link (green box), and add attach policies to it as bellow.

![_config.yml]({{ site.baseurl }}/images/role.png)

### Import default asset & files

To import the default scene asset, follow the *Re-Importing an Exported Sumerian Bundle* part of [this tutorial](https://www.andreasjakl.com/download-export-or-backup-amazon-sumerian-scenes-part-6/) by importing the [default asset pack](../download/sumerianhostrecognition-bundle.zip).

This scene will also require some [files](../download/filesToS3.zip) to works. To do that, go to [S3](https://console.aws.amazon.com/s3/) and create a new bucket and upload the script and img folder. Make sure to make both folder public to allow Sumerian to access them.

![_config.yml]({{ site.baseurl }}/images/public.png)

By default, the scene contains the following elements :

- CamButton and Micro : HTML entities representing the two buttons
- Webcam and Dialog : 3DHTML entities representing the camera display and dialog output
- Cristine : The Sumerian host

And the following scripts : 

- HostScript : Initialize all event on the scene
- Recognition : Recognition functions
- SwitchOnWebcamScript/SwitchOffWebcamScript : Functions activating/deactivating the webcam feed.

Now that the basic asset is configured on the scene, we will start by implementing the vocal interaction with the host using Lex.

# Chatbot with Lex

### Create the Lex chatbot

In our case, the Lex chatbot will be used mainly to ask the Sumerian host to switch on and off the webcam in order to start the facial detection/recognition.

To create the chatbot, go to your [Lex console](https://console.aws.amazon.com/lex) and follow these instructions :

1. On the *Bots* section, click on the *Create* button
2. Select *Custom* bot on the *create your bot* page
3. Customize your bot with a name, a voice matching with your host aspect... As bellow

![_config.yml]({{ site.baseurl }}/images/botCreation.png)

Once your bot is created, the next step is to create an intent representing a particular goal that the user wants to achieve by talking with the chatbot. Just click on the *create Intent* button and name it (e.g ChangeCameraStatus in our case).

Now we want to catch the webcam state the user want to change. This webcam state can be handled by a slot. Click on the + button next to *Slot types* and configure the slot by assigning to it a name and two values : On and Off. Then click on the *add slot to Intent* button.

![_config.yml]({{ site.baseurl }}/images/createSlot.png)

Finally, It remains only to configure the chatbot by adding a behaviour to the intent :$

1. Add the slot to the intent, assign a name...
2. Add utterances using the slot name
3. Add responses when the bot receive the user request

![_config.yml]({{ site.baseurl }}/images/intentAndSlotConf2.png)

### Configure the Sumerian host

# Webcam

# Recognition
