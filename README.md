# team4-mosquitto

# 0. Abstract
**Notion : [UVC Team 4 Project MQTT Smart Connector](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3)**  
**This UVC Team Project aims to build and implement Smart Factory Management System.**
**This document only notes on how I used a Smart Connector C# Program to connect to a PLC Edukit.**

I referred most of the codes from UVC, especially on Socket Connection.

## Index  

[**1. File Structure**](https://github.com/shlee9605/team4-mosquitto#1-file-structure)  
[**2. Development Environment**](https://github.com/shlee9605/team4-mosquitto#2-develop-environment)  
[**3. Getting Packages**](https://github.com/shlee9605/team4-mosquitto#3-getting-packages)  
[**4. Setting Configuration**](https://github.com/shlee9605/team4-mosquitto#4-setting-configuration)  
[**5. Concepts Used**](https://github.com/shlee9605/team4-mosquitto#5-used-concept)  
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

# 2. Development Environment

Worked on: Windows(Visual Studio 2022), PLC(Edukit)  
Used: .NET Core 3.1, MQTT, SocketI/O, ETC  
**You would need to refer to [My Notion(in Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3) for the Plugin Packages Description of this Project.**  

## Setup
For Development OS, I used `Windows10`.  
First, set up a working space for C#.
```console
> cd C:\Workspace
```
  
## Installation & Create Project & Run Project
You need to set the working environment up on Visual Studio 2022.  
Check [My Notion(in Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3) out on how to install it on Windows.  
  
  
# 3. Getting Packages

## Packages
```C#
.NET CORE == 3.1  
using uPLibrary.Networking.M2Mqtt == 1.1.0  
```
  
## .NET Core

### Installation
This library is for the C# Embedded Library.  
Check My Notion for procedures.
After installing it, you can use them just by importing it with commands.
The same applies to other packages.
  
## uPLibrary.Networking.M2Mqtt  

### Installation  
[Installation](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#ce40c0ed5b9845f29710f1d97575e3ab)  
  
### Configuration
To use MQTT in C#, you would need to specify HOST, PORT, PATH and TOPIC.  
Below shows how I did for this Project.  

```C#
int mqttport = Int32.Parse(edgeConfigResult.MqttBrokerPort);
mqttClient = new MqttClient(edgeConfigResult.MqttBrokerIP, mqttport, false, null, null, MqttSslProtocols.TLSv1_2);
```
  
### Usage
After configuring the MQTT broker, you can Publish & Subscribe through TOPIC.  
  
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
Create a `.gitignore` file in `C:\Workspace`.
Then set your files as follows in `C:\Workspace\.gitignore`.
  
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

We have already discussed about MQTT Configuration above.  
Though, there are some additional configurations that are needed to be done.  
Below code will show some data you need to configure in `C:\Workspace\dev.py`,  

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
  
  
# 5. Concepts Used
There aren't much Idea & Concepts used here that are needed to be mentioned here.
Check [Notion(Kor)](https://www.notion.so/1d50eee57be542fd8435cf5088dd9936#4a7257e9d60b4ae6b9ff57869f8b05f3) out instead.  
  
  
# 6. Usage Example
  
![image](https://user-images.githubusercontent.com/40204622/208420612-9870a88c-88a1-4b4c-9f7d-438adec068fa.png)  
  
