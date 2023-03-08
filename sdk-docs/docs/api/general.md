# API Overview

## The Theory
MuseBox Server works via asynchronous messaging API based on different protocols.
Every MuseBox Server implementation has an API system that supports 2  protocols:

- [ZeroMQ](https://zeromq.org/) (aka ZMQ or ØMQ or 0MQ), an embedded library for queue-based communication stack.

- [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), a standard two-way interactive communication protocol


You can choose the communication protocol when you will start the MuseBox server, you simply pass to the second parameter of the executable the protocol (`zmq` or `websocket`). For example:

```
./MuseBoxAudio websocket
```

If you need a point-to-point communication with 1 client or you have a web-based client, it is preferrable the WebSocket communication, otherwise is preferrable ZeroMQ. The message payload is the same in both protocols.

The communication is based on the Publish - Subscribe classic pattern. 
The Senders of messages, called publishers (in our case, we can call it "MuseBox Client"), do not program the messages to be sent directly to specific receivers, called subscribers. Messages are published without the knowledge of what or if any subscriber of that knowledge exists.

When MuseBox starts, it exposes its communication socket as a Subscriber. When it receives a message from a MuseBox Client, it responds to the Publisher at the socket that the MuseBox Client exposes. In order to do that, the MuseBox Client sends to the MuseBox Server in the message the address where it expects to receive the answer.
The payload of the message is in JSON format.

Let's see this example:

- A MuseBox Client sends this message to the MuseBox Server:

```
{
    "topic": "FaceDetection",
    "image": [...],
    "publisherQueue": "tcp://*:5000"
}
```

- the MuseBox Server receives the message. If the selected protocol is zeromq, according to the field "publisherQueue" will open a socket to the address `tcp://*:5000` and will send the response there.

- the MuseBox Server sends the reponse to the MuseBox Client: 
```
{
    "data":
        [
            {
                "face_BB":
                    {
                        "height":62,
                        "width":51,
                        "x":262,
                        "y":97
                    }
            }
        ],
    "publisherQueue":"tcp://*:5000",
    "status":"success",
    "topic": "FaceDetection"
}
```

**NB**: in this documentation, the optional parameters will be written with this syntax:
`parameter?: <value>`
* * *
* * *

## APIs

The MuseBox Server listens in localhost (127.0.0.1) and in any IP (0.0.0.0) at port 9696.

In WebSocket binding, the client simply must open a connection to `127.0.0.1:9696`, otherwise if the client is extern to the board, the connection string depends on the board's IP. For example, if the board has the IP `192.168.1.5`, the connection string is `192.168.1.5:9696`.
In ZMQ socket binding, the Client will bind at this address for sending data to MuseBox Server: `tcp://*:9696`

All the APIs are in JSON format, and have the same structure. The structure is the following:

```
{
    "topic": <string>,
    "only_face": <bool>,
    "image"/"input: <base64 image>/double array,
    "publisherQueue": <string>
}
```
Where:

- "topic" is the name of the Machine Learning task
- "image" is the OpenCV image in base64 format (string encoded in UTF8)
- "publisherQueue" is the address where the Client expects to receive the response
- "only_face" (optional) if you want to execute only the selected task and not the dependent tasks. For example: if you want to execute the face recognition task, you need to execute before that the face detection for extracting the sub-image that contains the face of the person that you want to recognize; with `only_face: true` you are sure that the passed image contains only the cropped face, meanwhile without the field `only_face`, MuseBox Server executes face detection and face recognition.

The MuseBox server response structure is the following:
```
{
    "data":
        [
            {
                "<topic>_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
            }
        ],
    "publisherQueue": <string>,
    "status": <string>,
    "topic": <string>
}
```
Where:

- "publisherQueue" is the same string that the Client sent previously

- "status" is the status of the request (success / error)

- "topic" is the requested inference task

- "data" is the inference result in JSON Array, where:

  - "\<topic>_BB" is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)

NB: if the selected task has dependent tasks (e.g. Face Recognition requires Face Detection) and you have not setted `only_face: true`, the field "data" contains all the dependent task executed.
For example, Face Recognition without `only_face: true`:

```
{
    "data": {
        "FaceDetection": {
            "data": [
                {
                    "face_BB": {
                        "height": 71,
                        "width": 65,
                        "x": 292,
                        "y": 56
                    }
                }
            ],
            "status": "success",
            "topic": "FaceDetection"
        },
        "FaceRecognition": [
            {
                "clientId": "1",
                "personFound": "unknown",
                "status": "success",
                "topic": "FaceRecognition"
            }
        ]
    }
}
```

* * *


### Face Analysis

#### **Face detection**

It detects the faces in a scene up to 32×32 pixel.

Request from Client:
```
"topic": "FaceDetection"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "face_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
            }
        ],
}
```
- face_BB is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)

#### **Face landmarking**

Given a face bounding box, it extracts the 98 relevant points of a face. 

Request from Client:
```
"topic": "FaceLandmark"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "landmarks": [196 int]
            }
        ]
}
```
- landmarks corresponds to (x,y) couples for face landmark points

#### **Face recognition**

Given a face bounding box and a database of faces, it recognizes a face in a scene.

You need to populate the directory `/usr/local/bin/database/people` with the photos of the desired people.

Request from Client:
```
"topic": "FaceRecognition"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "personFound": <string>
            }
        ]
}
```
- personFound corresponds to the name of the person


#### **Glasses detection**

Given a face bounding box, it detects whether a person is wearing glasses or not.

Request from Client:
```
"topic": "GlassesDetection"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "glasses": "glasses"/"no glasses"
            }
        ]
}
```
- glasses is the result of inference (glasses / no glasses)

#### **Age Detection**

Given a face bounding box, it determines the age of a person.

Request from Client:
```
"topic": "AgeDetection"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "age": <int>
            }
        ]
}
```
- age corresponds to the age in an integer number between 0 - 100


#### **Gender Detection**

Given a face bounding box, it determines if the person is female or male.

Request from Client:
```
"topic": "GenderDetection"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "gender": "man"/"woman"
            }
        ]
}
```
- gender corresponds to the gender of the person (man or woman)


* * *


#### **Emotion Recognition**

Given a face bounding box, it determines the facial expression/emotion of the person.

Request from Client:
```
"topic": "EmotionRecognition"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "emotion": "angry"/"disgust"/"fear"/"happy"/"sad"/"surprise"/"neutral"
            }
        ]
}
```
- emotion corresponds to the facial emotion of the person


* * *


### People Analysis

#### **People detection**

It detects the people in a scene up to 64×64 pixel.

Request from Client:
```
"topic": "PeopleDetection"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "person_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
            }
        ],
}
```

- people_BB is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


#### **Human Segmentation**

It segments the people present in a scene

Request from Client:
```
"topic": "HumanSegmentation"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "image":[ 320x320 double]
            }
        ],
}
```

- image is an array of doubles that creates the mask to be applied on top of the original image 

* * *


#### **Image Portrait**

It draws a black and white portrait of the people in the image.

Request from Client:
```
"topic": "ImagePortrait"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "image":[ 512x512 double]
            }
        ],
}
```

- image is an array of doubles that creates a black and white portrait image

* * *

### **Object Analysis**

#### **Object detection**

It detects the objects in a scene up to 64×64 pixel.

Request from Client:
```
"topic": "ObjectDetection"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "object_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
            }
        ],
}
```

- object_BB is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)

#### **Logo detection**

It detects the objects in a scene up to 64×64 pixel.

Request from Client:
```
"topic": "LogoDetection"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "logo_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
            }
        ],
}
```

- object_BB is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


#### **Logo Recognition**

Given a logo bounding box, it determines the brand name.

You need to populate the directory `/usr/local/bin/database/logos` with the photos of the desired logos.

Request from Client:
```
"topic": "LogoRecognition"
"image": "base64 image"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "logoFound": <string>
            }
        ]
}
```

- logoFound corresponds to the name of the logo


* * *

#### **Text Detection**

Given an image, it detects the texts inside of it.

Request from Client:
```
"topic": "TextDetection"
"image": "base64 image"
```

Response from Server:
```
{
    "data":
        [
            "text_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
        ]
}
```

- text_BB is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


* * *

#### **Monocular Depth**

Given an image in input, it returns its depth estimation.

Request from Client:
```
"topic": "MonoDepth"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "image":[ 128x128 double]
            }
        ],
}
```

- image is an array of doubles that represents the depth estimation of the input

* * *

#### **Monocular Depth #2**

Given an image in input, it returns its depth estimation.

Request from Client:
```
"topic": "MonoDepth2"
"image": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "image":[ 192x640 double]
            }
        ],
}
```

- image is an array of doubles that represents the depth estimation of the input

* * *


#### **Stochastic Difference**

Given two images in input, it returns how similiar those images are.

Request from Client:
```
"topic": "StochasticDifference"
"image": "base64 image"
"image2": "base64 image"
```
Response from Server:
```
{
    "data":
        [
            {
                "result": double
            }
        ],
}
```

- result is a double which value is a number from [0.0 - 1.0] describing how similiar the images in input are

* * *


### **Medical Analysis**
#### **Medical Detection**

Given an endoscopic image, it detects anything suspicious in it.

Request from Client:
```
"topic": "MedicalDetection"
"image": "base64 image"
```

Response from Server:
```
{
    "data":
        [
            "medical_BB":
                    {
                        "height": <int>,
                        "width": <int>,
                        "x": <int>,
                        "y": <int>
                    }
        ]
}
```

- medical_BB is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


* * *

#### **Medical Segmentation**

Given an endoscopic image, it segments anything suspicious in it.

Request from Client:
```
"topic": "MedicalSegmentation"
"image": "base64 image"
```

Response from Server:
```
{
    "data":
        [
            "countour":
                    {
                        "Point": {
                            "x": <int>,
                            "y": <int>
                        }
                    }
        ]
}
```

- data contains all the points of the countours of the task, according to OpenCV format (Point: x, y)


* * *


### **Audio Analysis**
#### **Zero Crossing Rate**

Given a double array in input, it measures how many times the waveform crosses the zero axis.

Request from Client:
```
"topic": "ZeroCrossingRate"
"input": [4410 double]

```

Response from Server:
```
{
    "data":
        [
            "result": [double, double]
        ]
}
```

- the first element of result is the number of times that the waveform crosses the zero axis, meanwhile the second one is the rate of the number of times that the waveform crosses the zero axis


* * *

#### **Signal Energy**

Given a double array in input, it returns the total magnitude of the signal.
For audio signals, that roughly corresponds to how loud the signal is.

Request from Client:
```
"topic": "SignalEnergy"
"input": [4410 double]
```

Response from Server:
```
{
    "data":
        [
            "result": double
        ]
}
```

- result is a double representing the total magnitude of the signal


* * *

#### **FFT**

Given a double array in input, it returns the FFT of the input.

Request from Client:
```
"topic": "FFT"
"input": [2048 double]
```

Response from Server:
```
{
    "data":
        [
            "result": [ 2048 double]
        ]
}
```

- result is the number of the given FFT of the input


* * *

#### **STFT**

Given a double array in input, it returns the STFT of the input.

Request from Client:
```
"topic": "STFT"
"input": [2048 double]
```

Response from Server:
```
{
    "data":
        [
            "result": [ 2048 double]
        ]
}
```

- result is the number of the given STFT of the input


* * *

#### **Mono Audio to Midi**

Given a double array in input, it returns the corresponding note in midi notation (from A0 to B0 of a monophonic audio, for example, if you pass to the function a sine wave of 440hz, the function returns A3).

Request from Client:
```
"topic": "Monoaudio2Midi"
"input": [5512 double]
```

Response from Server:
```
{
    "data":
        [
            "result": [string, string]
        ]
}
```

- the result is the corresponding note in midi notation of the input


* * *