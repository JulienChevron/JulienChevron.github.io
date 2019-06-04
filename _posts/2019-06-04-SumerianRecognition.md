---
layout: post
title: Sumerian concierge with facial recognition


---

This tutorial will show you how to create a basic virtual host on AWS Sumerian, able to discuss, recognize you and detect your emotion.

# Introduction

![_config.yml]({{ site.baseurl }}/images/Demo_scene2.png)

### Features

The goal of this tutorial is to create a scene with a host who can listen and interpret your words. 

By pressing the Space bar or the Microphone button, you will be able to record your voice a make the host understand what you said.

The host will be able to activate on request the webcam, in order to detect your emotions and recognize your face.

All this project will be realized on [AWS Sumerian](https://aws.amazon.com/fr/sumerian/) and using [AWS technologies](https://aws.amazon.com/).

### Prerequisites

- Webcam, Microphone and Speakers
- Sign in to Amazon Sumerian with your [AWS account](https://signin.aws.amazon.com/signin?client_id=signup&redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&page=resolve)
- Download the [default scene asset](../download/sumerianhostrecognition-bundle.zip)
- Download images and scripts [files](../download/filesToS3.zip)
- Be on point on [scripting](https://docs.sumerian.amazonaws.com/tutorials/create/beginner/scripting-basics/index.html) and [state machine](https://docs.aws.amazon.com/sumerian/latest/userguide/sumerian-statemachines.html) 

### Technologies used

- [Amazon Sumerian](https://aws.amazon.com/sumerian/) :  Used to create scene, handle user interaction and display a virtual host with which you can interact as a chat bot.
- [Amazon Lex](https://aws.amazon.com/lex/) : Service for building conversational interfaces using voice and text as a chat bot. This service is used to create intents for the Sumerian host and to react to the users requests.
- [Amazon Rekognition](https://aws.amazon.com/rekognition/) :  Image and video recognition service allowing facial recognition, emotion analysis.
- [Amazon DynamoDB](https://aws.amazon.com/lex/) : AWS database that will be used to save face ID and user's names.
- [Tracking.js](https://trackingjs.com/) : OpenCV based JavaScript library to detect faces on videos and images.

![_config.yml]({{ site.baseurl }}/images/tech.png)

# Init the scene

### Grant AWS access

First of all, you will need to set up your Sumerian scene by granting all AWS access you need in it. Therefore, create a Cognito Identity Pool by following [this tutorial](https://docs.sumerian.amazonaws.com/tutorials/create/beginner/aws-setup/). The Cognito identity Pool provides Sumerian users a temporary token allowing you to use AWS services from Sumerian such as Lex, Rekognition...

![_config.yml]({{ site.baseurl }}/images/cognito.png)

Once the Cognito Identity Pool is created by selecting the Polly and Lex template, you can get the Identity Pool ID (red box) and configure your Sumerian scene with it.

By default, you will only be able to access Lex and Polly. To add the Rekognition and DynamoDB access, click on the role link (green box), and add attach policies to it as below.

![_config.yml]({{ site.baseurl }}/images/role.png)

### Import default asset & files

To import the default scene asset, follow the *Re-Importing an Exported Sumerian Bundle* part of [this tutorial](https://www.andreasjakl.com/download-export-or-backup-amazon-sumerian-scenes-part-6/) by importing the [default asset pack](../download/sumerianhostrecognition-bundle.zip).

This scene also required some [files](../download/filesToS3.zip) to works. To do that, go to [S3](https://console.aws.amazon.com/s3/), create a new bucket and upload the script and img folder. Make sure to make both folder public to allow Sumerian to access them.

![_config.yml]({{ site.baseurl }}/images/public.png)

By default, the scene contains the following elements :

- **CamButton** and **Micro** : HTML entities representing the two buttons
- **Webcam** and **Dialog** : 3DHTML entities representing the camera display and dialog output
- **Cristine** : The Sumerian host

And the following scripts : 

- **HostScript** : Initialize all event on the scene
- **Recognition** : Recognition functions
- **SwitchOnWebcamScript**/**SwitchOffWebcamScript** : Functions activating/deactivating the webcam feed.

Now that the basic asset is configured on the scene, we will start by implementing the vocal interaction with the host using Lex.

# Chatbot with Lex

### Create the Lex chatbot

In our case, the Lex chatbot will mainly be used to ask the Sumerian host to switch on and off the webcam in order to start the facial detection/recognition.

Let’s get started on creating the chatbot by going to your [Lex console](https://console.aws.amazon.com/lex) and follow these instructions :

1. On the **Bots** section, click on the **Create** button
2. Select **Custom** bot on the **create your bot** page
3. Customize your bot with a name, a voice matching with your host aspect... As bellow

![_config.yml]({{ site.baseurl }}/images/botCreation.png)

The next step consists to create an intent representing a particular goal that the user wants to achieve by talking with the chatbot. Just click on the **create Intent** button and name it (e.g ChangeCameraStatus in our case).

Now, we want to catch the webcam state to which the user wants to go. This webcam state can be handled by a slot. Click on the **+** button next to **Slot types** and configure the slot by assigning to it a name and two values : On and Off. Then click on the **add slot to Intent** button.

![_config.yml]({{ site.baseurl }}/images/createSlot.png)

It only remains to configure the chatbot by adding a behaviour to the intent :

1. Add the slot to the intent, assign a name...
2. Add utterances using the slot name
3. Add responses to use when the bot receive the user request

![_config.yml]({{ site.baseurl }}/images/intentAndSlotConf2.png)

Finally, click on the **Build** button on the top of the page and wait the chat bot to be ready.

### Configure the Sumerian host

The Lex chatbot is now created but we want to use it as a vocal chatbot using the Sumerian host.

Begin with assigning a dialog component to the host. You just need to select your host entity on the **Sumerian Entities** panel, add a **Dialogue** component and configure it with your Lex bot name (as defined before) and set **$LATEST** as version.

![_config.yml]({{ site.baseurl }}/images/addDialog.png)

![_config.yml]({{ site.baseurl }}/images/dialogConf.png)

After the dialogue component is added, let's add a the behaviour below to the host entity.

![_config.yml]({{ site.baseurl }}/images/hostLexBehaviour.png)

This behaviour is working like this :

1. **AWS Ready** : Wait the AWS SDK to load
2. **Intro Speech** : Read an introduction speech to introduce the host
3. **Wait Microphone** : Wait the Space key to be down or the *microOn* message to be emitted (when the microphone button is pressed).
4. **Recording** : Contains a **start Microphone Recording** action to perform recording, emit the *startRecord* message (will be used to change the microphone button aspect) and wait the recording end when the Space key is up or when the *microOff* message is emitted (when the microphone button will be released).
5. **Recording finished** : Emit the *endRecord* message and contains a **stop Microphone Recording** action.
6. **Lex Processing** : Send the recorded message to Lex.
7. **Response & Lex error** : Only read the Lex response speech.
8. **End Response** : Emit the *endMessage* to reset the microphone button aspect. 

Finally, in order to catch all emitted message from this behaviour, and to add events on the microphone button, add a script component to the host and give to the script the two parameters required. These parameters are the link to the two image uploaded earlier in the S3 bucket and the intent and slot name to activate the webcam.

![_config.yml]({{ site.baseurl }}/images/addhostscript.png)

# Webcam

For the moment, you can ask the host to turn on and off the webcam, and the Lex chatbot will response you that the webcam state has been changed, but, nothing really happens.

To handle the Lex response and convert it to the action to change the camera state, you need to add the following code to the **initLexResponseEvent** function on the **hostScript*** file.

```js
ctx.onLexResponse = (data) => {
  if (data.dialogState === "Fulfilled") {
    if(data.intentName === args.intentCam){
      switch (data.slots[args.slotCam]){
        case "on":
          sumerian.SystemBus.emit("switchOn", true);
          break;
        case "off":
          sumerian.SystemBus.emit("switchOff", true);
          break;
        default:
          break;
      }
    }
  }
}
```

After that, the **onLexResponse** handler should detect when the user call the Lex intent to change the webcam state, and emit the good message (*switchOn* or *switchOff*).

Then, these two messages must be received by a new behaviour. Create a new behaviour attached to the host entity and modify it as the one below.

![_config.yml]({{ site.baseurl }}/images/behaviorWebcam.png)

1. **Webcam off/on** : Respectively listen the message *switchOn* and *switchOff*.
2. **Switch on/off** : Respectively execute the script **SwitchOnWebcamScript** and **SwitchOffWebcamScript**.
3. **Change Recognition State** : Execute the **RecognitionScript** (we will configure it later on).

The scripts activating the webcam use the [WebRTC API](https://webrtc.github.io/samples/) to stream the webcam feed on the webcam 3DHTML entity. Once the webcam is on, the Sumerian context variable ctx.worldData.cameraOn is set to true to notify the whole program that the camera is on.

```javascript
function switchOnWebcam(ctx){
  const video = document.getElementById("video");
  navigator.getMedia = (navigator.getUserMedia ||
      navigator.webkitGetUserMedia ||
      navigator.mozGetUserMedia ||
      navigator.msGetUserMedia);
  navigator.getMedia({
    video: true,
    audio: false
  }, function(stream) {
    let vendorURL = window.URL || window.webkitURL;
    video.srcObject=stream;
    video.play();
  }, function(error) {
    console.log(error);
  });
  ctx.worldData.cameraOn = true;
  ctx.worldData.pressButton("CamButton");
}
```

If everything went well, both when you click on the camera button or ask the host to activate the camera state, the webcam feed will be displayed on the webcam entity.

If you encounter any problem such as an orange webcam feed, the host never understanding your sentences... make sure you have given the browser permissions to access the microphone and webcam.

# Recognition

The final part consists to use the webcam feed to detect your face and call AWS Rekognition to detect your emotion and recognize you.

To sum up the system, when the webcam is turned on, the recognition script create a [JavaScript interval](https://www.w3schools.com/jsref/met_win_setinterval.asp) that make a screenshot of the webcam feed into a canvas, detect faces on this canvas and if a face is detected, call the AWS Rekognition service to detect emotion and recognition.

![_config.yml]({{ site.baseurl }}/images/recoDiagram.png)

### Create the recognition system

The recognition system isn't complicated to implement. Just follow [this tutorial](https://aws.amazon.com/blogs/machine-learning/build-your-own-face-recognition-service-using-amazon-rekognition/) to create the system and then feed the recognition collection with some pictures.

Keep in mind the **collection ID** and the name of the **DynamoDB table name** containing the face ID and the users names for further steps.

### Implement the detection

Start this step by adding the **recognitionScript** to the host entity and configure it with the collection ID, the dynamoDB table created right before and defined the JavaScript interval time (by default at 2 seconds).

![_config.yml]({{ site.baseurl }}/images/addRecoScript.png)

The facial detection is performed by the **Tracking.js** library, so, the script need to include it to make it works. Open the **recognitionScript** file in the editor and add the link to the **tracking-min.js** and **face-min.js** file uploaded in your S3 bucket, as external resources (without the HTTPS protocol part).

![_config.yml]({{ site.baseurl }}/images/addExtRes.png)

Before starting to implement the facial detection, take a look to the script while it's opened, it already contains all functions required to do the facial recognition using AWS Rekognition.

In the setup function, an object called “**tracker**” is created by Tracking.js and is configured to detect faces on videos or images.

```js
tracker = new tracking.ObjectTracker('face');
```

You can define the action performed by the tracker when the facial detection is called by creating an handler when the tracker emit the message *track*. If the data received on this handler doesn’t contain anything, no faces are detected. On the other hand, the tracker detected a face. In this case, the facial recognition and emotion detection can be called. 

Once the handler is defined, it only thing to do is to create the interval to call this detection at a predefined regular time interval. At every execution of the interval function, a screenshot of the webcam feed into the canvas is made, the facial detection is asked and the detection is stopped right after to avoid overloading the application.

To preform all these actions, copy the code below into the enter function (the function called when the script is executed from the behaviour).

```js
if(Boolean(ctx.worldData.cameraOn)){
  tracker.on('track', function(event) {
    if (event.data.length === 0) {
      output.innerHTML = "";
      resetCurrentState();
    } else {
      let img = getImageFromCanvas(canvas);
      imageRecognition(img, output, args.collectionID, args.dbTable, ctx);
    }
  });
  interval = setInterval(function(){
    drawVideoOnCanvas(video, canvas, 500, 500);
    let task = tracking.track("#canvas", tracker);
    task.stop();
  }, args.frameRate);
  ctx.transitions.success();
}else{
  cleanup(args, ctx);
  ctx.transitions.failure();
}
```

To finish, in order to deactivate the detection if the webcam is turned off, copy the following code in the cleanup function to clear everything when the recognition stops.

```js
if(interval != null){
  var output = document.getElementById(args.output);
  output.innerHTML = "";
  tracker.removeAllListeners();
  clearInterval(interval);
  interval = null;
  resetCurrentState();
}
```

Now, the system is ready to work ! Just start the Sumerian scene, ask the host to switch on the webcam, and the host should be able to detect you if your face is known on the recognition collection.

If you want to learn more about how the facial recognition and emotion detection actually works, read the next part.

### How AWS Rekognition works

All AWS services calls use asynchronous request. In order to synchronize the operations, the request results are casted into [Promise](https://javascript.info/promise-basics) (result of an asynchronous request operation). 

##### Emotion detection

The emotion detection uses the [detectFaces](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Rekognition.html#detectFaces-property) function of AWS Rekognition taking in parameter the binary image to analyze. The result is cast to Promise and contains faces information such as it size, emotions detected… 

```js
function detectFace(img){
  const params = {
    Image: {
      'Bytes': img
    },
    Attributes : [
      "ALL"
    ]
  };
  let request = rekognition.detectFaces(params);
  let promise = request.promise();
  return promise;
}
```

To get the face emotion, parse the emotion array returned by the **detectFace** function above and return the emotion with the maximum confidence.

```js
let max = 0;
let maxEmotion = "unknown";
let array = promise.FaceDetails[0].Emotions;
array.forEach(array => {
  if (array.Confidence > max) {
    max = array.Confidence;
    maxEmotion = array.Type.toLowerCase();
  }
});
```

The most detected emotion is now stored in the variable * maxEmotion *.

##### Facial recognition

The facial recognition function calls once again AWS Rekognition with the function [searchFacesByImage] (https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Rekognition.html#searchFacesByImage-property). This function takes the form of the binary image, the accuracy and the collection ID.

```js
function facialRecognition(img, collection, threshold, maxFaces){
  const params = {
    CollectionId: collection,
    Image: {
      'Bytes': img
    },
    FaceMatchThreshold: threshold,
    MaxFaces: maxFaces
  };
  let request = rekognition.searchFacesByImage(params);
  let promise = request.promise();
  return promise;
}
```

If the promise contains a detected face, you can get the detected Face ID with the following instruction.

```js
 if (promise.FaceMatches.length > 0) {
  const faceID = promise.FaceMatches[0]["Face"]["FaceId"];
}
```

##### Matching the face ID and the user name

To get the name associated with the Face ID, the script make a request to a DynamoDB table containing all Face ID and name of the Rekognition collection by calling the function [getItem](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#getItem-property) from AWS DynamoDB with the face ID and the table to look on as parameters. 

```js
function getNameWithFaceID(faceID, table){
  const params = {
    Key: {
      "RekognitionId": {
        S: faceID
      }
    },
    TableName: table
  };
  let request = dynamodb.getItem(params);
  let promise = request.promise();
  return promise;
}
```

Finally, the name  received by the DynamoDB request is obtained by this instruction. 

```js
name = promise.Item.Fullname.S;
```

When all recognition information are filled, the script only need to display the result on the **Dialog** 3DHTML entity and make the host speak your name. To modify the host speech, just change the body of the first speech component and play the speech.

```javascript
function modifySpeech(text, ctx) {
  let speech = ctx.entity
                   .getComponent("speechComponent")
                   .speeches[0];
  speech.body = '<speak>'+ text + '</speak>';
  speech.play();
  speech.body = '';
}
```

# Conclusion

You’ve added everything you need to get a virtual host able to recognize your face and emotions. Play your scene and open the debugger console from your browser to see if there are any errors.

To talk with the host, you can either hold the space bar or the microphone button down while your speaking.

You can now easily enhance the system by adding other features such as different greetings depending on the emotion detected, add other intents to the Lex chatbot...

Thank you for reading this tutorial.

