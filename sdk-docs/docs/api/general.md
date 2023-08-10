# API Overview

## The Theory
MuseBox Server works via asynchronous messaging API based on different protocols.
Every MuseBox Server implementation has an API system that supports 2 protocols:

- [ZeroMQ](https://zeromq.org/) (aka ZMQ or ØMQ or 0MQ), an embedded library for queue-based communication stack.

- [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), a standard two-way interactive communication protocol


You can choose the communication protocol when you start the MuseBox server, by simply passing as a second parameter of the executable the protocol (`zmq` or `websocket`). For example:

```bash
./MuseBoxAudio websocket
```

If you need a point-to-point communication with 1 client or you have a web-based client, it is preferrable to use the WebSocket communication, otherwise it is preferrable to use ZeroMQ. The message payload is the same in both protocols.

The communication is based on the Publish - Subscribe classic pattern. 
The Senders of messages, called publishers (in our case, we can call it "MuseBox Client"), do not program the messages to be sent directly to specific receivers, called subscribers. Messages are published without the knowledge of what or if any subscriber of that knowledge exists.

When MuseBox starts, it exposes its communication socket as a Subscriber. When it receives a message from a MuseBox Client, it responds to the Publisher at the socket that the MuseBox Client exposes. In order to do that, the MuseBox Client sends to the MuseBox Server in the message the address where it expects to receive the answer.
The payload of the message is in JSON format.

Let's see this example:

- A MuseBox Client sends this message to the MuseBox Server:

```json
{
    "topic": "FaceDetection",
    "image": [...],
    "publisherQueue": "tcp://*:5000"
}
```

- the MuseBox Server receives the message. If the selected protocol is zeromq, according to the field "publisherQueue", it will open a socket to the address `tcp://*:5000` and it will send the response there.

- the MuseBox Server sends the reponse to the MuseBox Client: 
```json
{
    "data":
        [
            {
                "boundingBox":
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

```bash
{
    "topic": <string>,
    "image"/"input": <base64 image>/<double array>,
    "publisherQueue": <string>
}
```
Where:

- `topic` is the name of the Machine Learning task
- `image` is the OpenCV image in base64 format (string encoded in UTF8)
- `input` is an array of doubles
- `publisherQueue` is the address where the ZMQ Client expects to receive the response

The MuseBox server response structure is the following:
```json
{
    "data":
    [
        {
            "boundingBox":
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

- `publisherQueue` is the same string that the Client sent previously

- `status` is the status of the request (success / error)

- `topic` is the requested inference task

- `data` is the inference result in JSON Array, where:

- `boundingBox` is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)

* * *


### Face Analysis

#### Face Detection

It detects the faces in a scene up to 32×32 pixel.

Request from Client:
```json
"topic": "FaceDetection"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": [
        {
            "boundingBox": {
                "height": <int>,
                "width": <int>,
                "x": <int>,
                "y": <int>
            }
        }
    ]
}
```
- `boundingBox` is one of the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)

#### Face Recognition

Given a face bounding box and a database of faces, it recognizes a face in a scene.

You need to populate the directory `/usr/local/bin/database/people` with the photos of the desired people.

Request from Client:
```json
"topic": "FaceRecognition"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": <string>
    }
}
```
- `prediction` corresponds to the name of the person

#### Face Landmark

Given a face bounding box, it extracts the 98 relevant points of that face. 

Request from Client:
```json
"topic": "FaceLandmark"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": [196 float]
    }
}
```
- `prediction` corresponds to (x,y) couples for face landmark points

#### Glasses Detection

Given a face bounding box, it detects whether a person is wearing glasses or not.

Request from Client:
```json
"topic": "GlassesDetection"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": "glasses"/"no glasses"
    }
}
```
- `prediction` is the result of inference (glasses / no glasses)

#### Age Detection

Given a face bounding box, it determines the age of a person.

Request from Client:
```json
"topic": "AgeDetection"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": <int>
    }
}
```
- `prediction` corresponds to the age in an integer number between 0 - 100


#### Gender Detection

Given a face bounding box, it determines if the person is female or male.

Request from Client:
```json
"topic": "GenderDetection"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": "man"/"woman"
    }
}
```
- `prediction` corresponds to the predicted gender of the person (man or woman)


* * *


#### Emotion Recognition

Given a face bounding box, it determines the facial expression/emotion of the person.

Request from Client:
```json
"topic": "EmotionRecognition"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": "angry"/"disgust"/"fear"/"happy"/"sad"/"surprise"/"neutral"
    }
}
```
- `prediction` corresponds to the facial emotion of the person


* * *


### People Analysis

#### People Detection

It detects the people in a scene up to 32×32 pixel.

Request from Client:
```json
"topic": "PeopleDetection"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": [
        {
            "boundingBox": {
                "height": <int>,
                "width": <int>,
                "x": <int>,
                "y": <int>
            }
        }
    ]
}
```

- `boundingBox` is one of the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


#### Human Segmentation

Human Segmentation segments the people present in a scene

Request from Client:
```json
"topic": "HumanSegmentation"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": {
        "prediction": <string of [320x320 double]>
    }
}
```

- `prediction` is an array of doubles that creates the mask to be applied on top of the original image 

* * *


#### Image Portrait

Image Portrait draws a black and white portrait of the people in the image.

Request from Client:
```json
"topic": "ImagePortrait"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": [
        {
            "image": [512x512 double]
        }
    ],
}
```

- `image` is an array of doubles that creates a black and white portrait image

* * *

### Object Analysis

#### Object Detection

Object Detection detects the objects in a scene up to 32×32 pixel.

Request from Client:
```json
"topic": "ObjectDetection"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": [
        {
            "boundingBox": {
                "height": <int>,
                "label": <string>,
                "width": <int>,
                "x": <int>,
                "y": <int>,
            }
        }
    ]
}
```

- `boundingBox` is one of the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)

#### Logo Detection

Logo Detection detects the objects in a scene up to 32×32 pixel.

Request from Client:
```json
"topic": "LogoDetection"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": [
        {
            "boundingBox": {
                "height": <int>,
                "width": <int>,
                "x": <int>,
                "y": <int>
            }
        }
    ]
}
```

- `boundingBox` is one of the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


#### Logo Recognition

Given a logo bounding box, it determines the brand name.

You need to populate the directory `/usr/local/bin/database/logos` with the photos of the desired logos.

Request from Client:
```json
"topic": "LogoRecognition"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": {
        "prediction": <string>
    }
}
```

- `prediction` corresponds to the name of the logo


* * *

#### Text Detection

Given an image, it detects the texts inside of that image.

Request from Client:
```json
"topic": "TextDetection"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": [
        {
            "boundingBox": {
                "height": <int>,
                "width": <int>,
                "x": <int>,
                "y": <int>
            }
        }
    ]
}
```

- `boundingBox` is one of the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


* * *

#### Monocular Depth

Given an image in input, it returns the image's depth estimation.

Request from Client:
```json
"topic": "MonoDepth"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": {
        "prediction": <string of [128x128 double]>
    }
}
```

- `prediction` is an array of doubles that represents the depth estimation of the input

* * *

#### Monocular Depth #2

Given an image in input, it returns the image's depth depth estimation.

Request from Client:
```json
"topic": "MonoDepth2"
"image": "base64 image"
```
Response from Server:
```json
{
    "data": {
        "prediction": <string of [192x640 double]>
    }
}
```

- `prediction` is an array of doubles that represents the depth estimation of the input

* * *


#### Stochastic Difference

Given two images in input, it returns a double number that indicates how similiar those images are.

Request from Client:
```json
"topic": "StochasticDifference"
"image": "base64 image"
"image2": "base64 image"
```
Response from Server:
```json
{
    "data": [
        {
            "prediction": double
        }
    ],
}
```

- prediction is a double which value is a number from [0.0 - 1.0] describing how similiar the images in input are, 0.0 means the images are the identical, meanwhile 1.0 means the images differ alot

* * *


### Medical Analysis
#### Medical Detection

Given an endoscopic image, it detects anything suspicious in it.

Request from Client:
```json
"topic": "MedicalDetection"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": [
        {
            "boundingBox": {
                "height": <int>,
                "label": <string>,
                "width": <int>,
                "x": <int>,
                "y": <int>
            }
        }
    ]
}
```

- `boundingBox` is the bounding boxes of the task, according to OpenCV format (from top-left to bottom-right)


* * *

#### Medical Segmentation

Given an endoscopic image, it segments anything suspicious in it.

Request from Client:
```json
"topic": "MedicalSegmentation"
"image": "base64 image"
```

Response from Server:
```json
{
    "data": [
        {
            "countour": [
                x1, y1, x2, y2, ... int
            ],
            "label": <string>
        }
    ]
}
```

- data contains all the points of the countours of the task, according to OpenCV format (Point: x, y) and its respective label



* * *


### Audio Analysis
#### Zero Crossing Rate

Given a double array in input, it measures how many times the waveform crosses the zero axis.

Request from Client:
```json
"topic": "ZeroCrossingRate"
"input": [4410 double]
```

Response from Server:
```json
{
    "data": {
        "prediction": [double, double]
    }

}
```

- the first element of `prediction` is the number of times that the waveform crosses the zero axis, meanwhile the second one is the rate of the number of times that the waveform crosses the zero axis


* * *

#### Signal Energy

Given a double array in input, it returns the total magnitude of the signal.
For audio signals, that roughly corresponds to how loud the signal is.

Request from Client:
```json
"topic": "SignalEnergy"
"input": [4410 double]
```

Response from Server:
```json
{
    "data": {
        "prediction": double
    }
}
```

- `prediction` is a double representing the total magnitude of the signal


* * *

#### FFT

Given a double array in input, it returns the FFT of the input.

Request from Client:
```json
"topic": "FFT"
"input": [2048 double]
```

Response from Server:
```json
{
    "data": {
        "prediction": [ 2048 double]
    }
}
```

- `prediction` is the number of the given FFT of the input


* * *

#### STFT

Given a double array in input, it returns the STFT of the input.

Request from Client:
```json
"topic": "STFT"
"input": [2048 double]
```

Response from Server:
```json
{
    "data": {
        "prediction": [ 2048 double]
    }
}
```

- `prediction` is the number of the given STFT of the input


* * *

#### Mono Audio to Midi

Given a double array in input, it returns the corresponding note in midi notation (from A0 to B0 of a monophonic audio, for example, if you pass to the function a sine wave of 440hz, the function returns A3).

Request from Client:
```json
"topic": "Monoaudio2Midi"
"input": [5512 double]
```

Response from Server:
```json
{
    "data": {
        "prediction": [string, string]
    }
}
```

- `prediction` is the corresponding note in midi notation of the input


* * *