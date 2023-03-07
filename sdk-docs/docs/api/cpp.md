# C++ Example

The C++ example is available on Github:
https://github.com/MakarenaLabs/MuseBox-client-examples/tree/main/Cpp

You can build the example directly on the board with this command:

```
bash -x build_client.sh
```

You can run the example on the board with this command:

```
./client <topic> <image | txt> [<image 2>]
```

`topic`: a machine learning task of the following: FaceDetection", "FaceRecognition", "FaceLandmark", "AgeDetection", "GenderDetection","GlassesDetection","EmotionRecognition","TextDetection", "LogoDetection", "LogoRecognition", "PeopleDetection", "ObjectDetection", "MonoDepth","MonoDepth2", "MedicalDetection", "MedicalSegmentation","HumanSegmentation", "ImagePortrait", "StochasticDifference", "ZeroCrossingRate", "SignalEnergy", "EnergyEntropy", "FFT", "STFT", "Monoaudio2Midi"

`image`: a local image path (like jpeg, png, jpg, etc image)
`txt`: a txt file that contains values
`image 2 (optional)`: a second local image path