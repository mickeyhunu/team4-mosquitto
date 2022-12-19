# team4-mosquitto

# 0. Abstract
**Notion : [UVC Team 4 Project MQTT Smart Connector](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3)**  
**This project is about UVC Team Project**  
**This project is to implement Smart Factory Management System.**  
This document only notes how I used Smart Connector C# Program   
to connect with PLC Edukit  

I Referenced most of the code from UVC,
Especially On Socket Connection Part.

## Index  

[**1. File Structure**](https://github.com/shlee9605/team4-mosquitto#1-file-structure)  
[**2. Develop Environment**](https://github.com/shlee9605/team4-mosquitto#2-develop-environment)  
[**3. Getting Packages**](https://github.com/shlee9605/team4-mosquitto#3-getting-packages)  
[**4. Setting Configuration**](https://github.com/shlee9605/team4-mosquitto#4-setting-configuration)  
[**5. Used Concept**](https://github.com/shlee9605/team4-mosquitto#5-used-concept)  
[**6. Usage Example**](https://github.com/shlee9605/team4-mosquitto#6-usage-example)  
  
  
# 1. File Structure

```
.
├── SmartConnector          
│   ├── Program.cs          # Main Program
│   ├── XGTAddressData.cs   # XGT Class Address data
│   ├── XGTClass.cs         # XGT Class
│   ├── XGTData.cs          # XGT Class Data
│   ├── SmartConnector      # project file
│   └── EdgeConfigFile.json # config
├── .gitignore
├── README
└── FlexingEdukit           # Solution file
```

# 2. Develop Environment

For : Windows(Visual Studio 2022), PLC(Edukit)  
Used : .NET Core 3.1, MQTT, SocketI/O, ETC  
**You Must Refer [My Notion(in Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3) Together for Plugin Packages Description of This Project**  

## Setup
For Development OS, I used `Windows10`.  
Then, set your working space for C#  
```console
> cd C:\Workspace
```
  
## Installation & Create Project & Running Project
You need to set up Visual Studio 2022  
Checkout [My Notion(in Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3) for Installation in Windows  
  
  
# 3. Getting Packages

## Packages
```C#
.NET CORE == 3.1  
using uPLibrary.Networking.M2Mqtt == 1.1.0  
```
  
## .NET Core

### Installation
This library is for C# Embedded Library.  
You need to check out My Notion instead.  
After Installing, You can use them just by importing with commands  
Same applies to other packages  
  
## uPLibrary.Networking.M2Mqtt  

### Installation  
[Installation](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#ce40c0ed5b9845f29710f1d97575e3ab)  
  
### Configuration
To use MQTT in C#, You need to specify HOST, PORT, PATH and TOPIC  
Below shows you how I configurated in this Project  

```C#
int mqttport = Int32.Parse(edgeConfigResult.MqttBrokerPort);
mqttClient = new MqttClient(edgeConfigResult.MqttBrokerIP, mqttport, false, null, null, MqttSslProtocols.TLSv1_2);
```
  
### Usage
After configuring MQTT broker, then you can Publish&Subscribe through TOPIC  
  
```C#
mqttClient.MqttMsgPublishReceived += PLCWrite;
string topic = edgeConfigResult.FrontId;
mqttClient.Subscribe(new string[] { topic }, new byte[] { MqttMsgBase.QOS_LEVEL_EXACTLY_ONCE });
...
string json = JsonConvert.SerializeObject(EduKitData, Formatting.Indented);
string data = json;  
string topic = edgeConfigResult.EdukitId;
try
{
  mqttClient.Publish(topic, Encoding.Default.GetBytes(data),
  MqttMsgBase.QOS_LEVEL_EXACTLY_ONCE, false);
}
catch (Exception ex)
{
  Console.WriteLine(ex.Message + "\n" + ex.StackTrace);
}
```
  
  
# 4. Setting Configuration
Create your `.gitignore` file in `C:\Workspace`, then setup like below.  
In `C:\Workspace\.gitignore`,  
  
```
.vs/
SmartConnector/EdgeConfigFile.json
SmartConnector/bin
SmartConnector/bin
SmartConnector/obj
SmartConnector/obj
...       # default .gitignore for C#  
```
  
## EdgeConfigFile.json configuration

We have already discuss about MQTT Configuration above.  
Though, there are some additional configurations that need to be done.  
Below code will show some data you need to configure  
In `C:\Workspace\dev.py`,  

```JSON
"EdukitId": "Your_Subscribe_TOPIC",
"EdukitIP": "Your_PLC_IP",
"EdukitPort": "Your_PLC_Port",
"FrontId": "Your_Publish_TOPIC",
"MqttBrokerIP": "Your_Broker_IP",
"MqttBrokerPort": "Your_Broker_Port",
"DelayTime": "100",
"DebugType": "Debug"
```
  
  
# 5. Used Concept
There aren't much Idea&Concept We need to discuss on this Git.  
Check [Notion(Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3) instead.  
  
  
# 6. Usage Example
  
![image](https://user-images.githubusercontent.com/40204622/208420612-9870a88c-88a1-4b4c-9f7d-438adec068fa.png)  
  
