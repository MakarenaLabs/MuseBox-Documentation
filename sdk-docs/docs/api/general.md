# API Overview

## The Theory
MuseBox Server works via asynchronous messaging API based on different protocols.
Every MuseBox Server implementation has a common API based on [ZeroMQ](https://zeromq.org/) (aka ZMQ or ØMQ or 0MQ), an embedded library for queue-based communication stack.

The communication is based on the Publish - Subscribe classic pattern. 
The Senders of messages, called publishers (in our case, we can call it "MuseBox Client"), do not program the messages to be sent directly to specific receivers, called subscribers. Messages are published without the knowledge of what or if any subscriber of that knowledge exists.

When MuseBox starts, exposes its communication socket as a Subscriber. When it receives a message from a MuseBox Client, it responds to the Publisher to the socket that the MuseBox Client exposes. In order to to that, the MuseBox Client send to the MuseBox Server message the address where it expects to receive the answer.
The payload of the message is in JSON format.

Let's see this example:

- A MuseBox Client send this message to the MuseBox Server:

```
{
    "topic": "FaceDetection",
    "image": [...],
    "imageHeight": 250,
    "imageWidth": 250,
    "publisherQueue": "tcp://*:5000"
}
```
- the MuseBox Server receives the message, and according to the field "publisherQueue", will open a socket to the address `tcp://*:5000` and will send the response there.
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
    "execution_time":4.957312,
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

The MuseBox Server listens in localhost (127.0.0.1) at port 9696 in TCP protocol. According to that, in ZMQ socket binding, the Client will bind at this address for sending data to MuseBox Server: `tcp://*:9696`

All the APIs are in JSON format, and have the same structure. The structure is the following:

```
{
    "topic": <string>,
    "only_face": <bool>,
    "image": <base64 image>,
    "imageHeight": <int>,
    "imageWidth": <int>,
    "publisherQueue": <string>
}
```
Where:

- "topic" is the name of the Machine Learning task
- "image" is the OpenCV image in base64 format (string encoded in UTF8)
- "imageHeight" is the height of the current image
- "imageWidth" is the width of the current image
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
    "execution_time": <double>,
    "publisherQueue": <string>,
    "status": <string>,
    "topic": <string>
}
```
Where:
- "execution_time" is the inference time on FPGA
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

It detects the faces in a scene up to 32×32 pixel

Request from Client:
```
"topic": "FaceDetection"
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

Given a face bounding box and a database of faces, it recognizes a face in a scene

You need to populate the directory `/usr/local/bin/database/people` with the photos of the desired people.

Request from Client:
```
"topic": "FaceRecognition"
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

Given a face bounding box, it detects whether a person wear glasses or not

Request from Client:
```
"topic": "GlassesDetection"
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

#### **Eye Blink Detection**

Given a face bounding box, it detects where are the eyes and it determines if they are blinking or not

Request from Client:
```
"topic": "EyeBlink"
"only_face"?: true
```

Response from Server:
```
{
    "data":
        [
            {
                "eyeState": "opened"/"closed"
            }
        ]
}
```
- eyeState is the result of inference (opened eyes / closed eyes)

#### **Age Detection**

Given a face bounding box, it determines the age of a person

Request from Client:
```
"topic": "AgeDetection"
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

Given a face bounding box, it determines if the person is female or male

Request from Client:
```
"topic": "GenderDetection"
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


### People Analysis

#### **People detection**

It detects the people in a scene up to 64×64 pixel

Request from Client:
```
"topic": "PeopleDetection"
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

* * *

### **Object Analysis**

#### **Object detection**

It detects the objects in a scene up to 64×64 pixel

Request from Client:
```
"topic": "ObjectDetection"
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

It detects the objects in a scene up to 64×64 pixel

Request from Client:
```
"topic": "LogoDetection"
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


#### **Logo Recognition**

Given a logo bounding box, it determines the brand name

You need to populate the directory `/usr/local/bin/database/logo` with the photos of the desired logos.

Request from Client:
```
"topic": "LogoRecognition"
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
