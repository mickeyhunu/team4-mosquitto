# team4-mosquitto

# 0. Abstract
**Notion : [UVC Team 4 Project Yolo Socket Stream](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#778988f111d4416abeb34f7d15eb9f94)**  
**This project is about UVC Team Project**  
**This project is to implement Smart Factory Management System.**  
This document only notes how I used Yolo Model & Socket System & Flask to apply  
AI Model & Socket Connection with PLC & Stream on HTTP Server  

I Referenced most of the code from UVC,
Especially On Socket Connection Part.

## Index  

[**1. File Structure**](https://github.com/shlee9605/team4-yolosocket#1-file-structure)  
[**2. Develop Environment**](https://github.com/shlee9605/team4-yolosocket#2-Develop-Environment)  
[**3. Getting Packages**](https://github.com/shlee9605/team4-yolosocket#3-Getting-Packages)  
[**4. Setting Configuration**](https://github.com/team4-yolosocketmd#4-Setting-Configuration)  
[**5. Used Concept**](https://github.com/shlee9605/team4-yolosocket#5-Used-Concept)  
[**6. Usage Example**](https://github.com/shlee9605/team4-yolosocket#6-Usage-Example)  
  
  
# 1. File Structure

```
.
└── dev.py        # Main
    ├── cam.py    # Socket Sending Camera
    └── stream.py # Streaming Camera
```

# 2. Develop Environment

For : Windows(anaconda prompt ver3)  
Used : Yolo, OpenCV, Flask, ETC  
**You Must Refer [My Notion(in Kor)](https://www.notion.so/UVC-c36970dd6c884131b159ea837790db94) Together for Plugin Packages Description of This Project**  

## Setup
For Development OS, I used `Windows10(anaconda3)`.  
Then, set your working space for python  
```console
> cd C:\Workspace
```
  
## Installation
You need to set up Yolov5
Checkout [My Notion(in Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#38e3ee19b7d34922bc082fc9921ff235) for Installation in Windows  
You can also refer to [Yolov5 documents](https://github.com/ultralytics/yolov5),  
which includes information about [Training Custom Dataset](https://github.com/ultralytics/yolov5/wiki/Train-Custom-Data), [Roboflow](https://roboflow.com/?ref=ultralytics)
  
## Activate Anaconda
Create Your Project
```console
> conda create -n yolov5 python=3.9  
> conda activate yolov5  
```
  
## Running Project
You can run your project via commands below

### default(debug Mode)
```console
> python dev.py  
```
  
# 3. Getting Packages

## Packages
```Python
import cv2
import torch
from time import sleep

from threading import Thread
from socket import *  

from flask import Flask  
from flask import Response  
from flask import stream_with_context 
```
  
## OpenCV

### Installation
The following allows you to use OpenCV library, 
which is mainly about vision sensing  

```console
> pip install opencv-python>=4.1.1
```  
  
### Configuration
For this project, You need to setup your own camera.  
Then, you need to check out which camera you are using from **Windows Device Manager**.  
After checking which camera you are using, You can Open your camera through python  
via below code.  
```Python
if cv2.ocl.haveOpenCL() :
    cv2.ocl.setUseOpenCL(True)
print('OpenCL : ', cv2.ocl.haveOpenCL())
webcam = cv2.VideoCapture(0)
```
  
### Usage
After making sure that your camera is opened,  
You can use variety of OpenCV functions.  
Below code is some example of how I used OpenCV in this project
```Python
detector = cv2.SimpleBlobDetector_create(params)    #Detecting Blob
...
imgRGB = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)     #Making RGB file to BGR for openCV
...
cv2.putText(imgRGB, 'Dice', (dice[0], dice[1]), cv2.FONT_HERSHEY_SIMPLEX, 2,(0, 255, 0), 2) #Putting Text
cv2.rectangle(imgRGB, (dice[0], dice[1]), (dice[2], dice[3]), (0, 0, 255), 3)               #Drawing Rectangle
...
frame_blurred = cv2.GaussianBlur(dst, Gaussian_ksize, 1)      #Gaussian Blur Filter
frame_gray = cv2.cvtColor(frame_blurred, cv2.COLOR_BGR2GRAY)    #Gray Filter
frame_canny = cv2.Canny(frame_gray, canny_threshold_min, canny_threshold_max, apertureSize=3, L2gradient=True)  #Canny Edge
...
```
  
## Torch

### Installation
This allows you to use Torch library, which means that you are now  
available to Use Trained Yolo .pt Model

```console
> pip install torch>=1.7.0
```  
  
### Configuration
To use yolo model and call the trained model successfully,  
You need to specify your yolo model's path

Below Shows how I specify My Yolo&Yolo model
```Python
...
model_label = torch.hub.load('../yolov5', 'custom', path='../yolov5/runs/train/dices5/weights/last.pt', source='local') # Read Train Model
...
```
  
### Usage
After reading yolo model via pytorch, You can use it via returned variable.
```Python
results = model_label(imgRGB)
results = results.pandas().xyxy[0][['name','xmin','ymin','xmax','ymax']]

dice = [0, 0, 0, 0]
cup = [0, 0, 0, 0]

for num, i in enumerate(results.values):
  if i[0] == 'Dice' and dice[0] == 0 :                
    dice = [int(i[1]), int(i[2]), int(i[3]), int(i[4])]
  if i[0] == 'Cup' and cup[0] == 0 :
    cup = [int(i[1]), int(i[2]), int(i[3]), int(i[4])]
  if dice[0] != 0 and cup[0] != 0:
    break
```
  
## Flask

### Installation
This allows you to open route server via python.
I used it in this project to stream my Yolo-Applied-Vision.  

```console
> pip install flask>=2.2.2  
```  

### Configuration
To Use Flask Model, You need to setUp both Host and Port.  
Here is the following to setup host and port using flask
  
```Python
...  
app = Flask(__name__)
@app.route('/stream')
...  
app.run(host='0.0.0.0', port=3002)  #you can see your stream through `http://localhost/stream`
```
  
### Usage
After setting up, you can stream your vision,  
using python project as server.  
Below you can see some example how I used in this project
```Python
def stream():    
  try :
    return Response(
      stream_with_context( streaming() ),
      mimetype='multipart/x-mixed-replace; boundary=frame' )
  except Exception as e :
    print('stream error : ',str(e))

def streaming():
  try : 
    while webcam.isOpened():
      success, frame = webcam.read()
      if not success:
        break
      else:
        frame = stream_yolo.yolo(frame, model_label)
        ret, buffer = cv2.imencode('.jpg', frame)
        frame = buffer.tobytes()
        yield (b'--frame\r\n'
          b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')
  except GeneratorExit :
    print( 'disconnected stream' )
```
  
## Thread, Time, Socket

### Installation
This packages are embedded libraries in Python.  
  
```Python
from socket import *
from threading import Thread
from time import sleep
```  
  
### Configuration
There Aren't Much to Configure.  
However, you do need to configure for successful socket connection.  
```Python
HOST = '192.168.0.120'  # Edukit Port
PORT = 2004
ADDR = (HOST,PORT)
```
  
### Usage  
Below each will show how I used those embedded library in this project.  
  
#### Thread  
```Python  
def server():
...
t=Thread(target=server, daemon=True)
t.start()
...
```  
  
#### Time  
```Python  
...
sleep(1)  #Because PLC cannot receive data frequently, it needs guaranteed 1 sec delay  
```    
  
#### Socket  
```Python  
...
clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect(ADDR)
print('Connection PLC Success!')
clientSocket.send(socketTxData + num_little)
clientSocket.close()
print('close PLC Success!')
```  
  
  
# 4. Setting Configuration
I have not used extra configuration file.
So, there aren't much for you to configure
  
## Flask configuration

We have already discuss about Flask Configuration above.  
In `C:\Workspace\dev.py`,  

```Python
...  
app = Flask(__name__)
@app.route('/stream')
...  
app.run(host='0.0.0.0', port=3002)  #you can see your stream through `http://localhost/stream`
```
  
Then, you can check your vision on http://(your IP)/stream
  
## Socket configuration
  
We have already discuss about Socket Configuration above.  
In `C:\Workspace\lib\cam.py`,  
```Python
HOST = '192.168.0.120'  # Edukit Port
PORT = 2004
ADDR = (HOST,PORT)
```
  
# 5. Used Concept
Most of the Concept are already specified in [Our Notion(Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#fda79f7cccda4e088c23260d622272f1).  
I am going to write the Big concept I used in this project Approximately,
Writing specifically Instead.

## Yolo
[Yolo Git](https://github.com/ultralytics/yolov5)  
[Our Notion(Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#e8e36a7d3e494eef94265ebc1264daa0)

## Image Pre-Processing
especailly about ***Guassian Blur, Gray Filter, Sobel Filter, Canny Edge***  
[Our Notion(Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#b4c2b136a9d2479aac508cf5c9a98d1e)
  
  
# 6. Usage Example
  
![1](https://user-images.githubusercontent.com/40204622/208363538-d8618519-d78c-46bd-a30a-0bfccab3f22a.PNG)
  
