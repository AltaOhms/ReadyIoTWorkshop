# Windows 10 IoT + Azure IoT Edge - Advance Lab (60 Min)

## Prerequisites

- An instance of IoT Hub from the previous HOL
- Dot Net Core 2.2
https://dotnet.microsoft.com/download/thank-you/dotnet-sdk-2.2.107-windows-x64-installer

## Key Components

## Step 1: Register a new Azure IoT Edge device


### Sign in to access your IoT hub

You can use the Azure IoT extensions for Visual Studio Code to perform operations with your IoT hub. For these operations to work, you need to sign in to your Azure account and select the IoT hub that you are working on.

1. In Visual Studio Code, open the **Explorer** view.

2. At the bottom of the Explorer, expand the **Azure IoT Hub Devices** section.

   ![Expand Azure IoT Hub Devices section](./images/Register-device-vscode/azure-iot-hub-devices.png)

3. Click on the **...** in the **Azure IoT Hub Devices** section header. If you don't see the ellipsis, click on or hover over the header.

4. Choose **Select IoT Hub**.

5. If you are not signed in to your Azure account, follow the prompts and sign in to your Azure account.

6. Select your Azure subscription.

7. Select your IoT hub.

### Create a device

1. In the VS Code Explorer, expand the **Azure IoT Hub Devices** section.

2. Click on the **...** in the **Azure IoT Hub Devices** section header. If you don't see the ellipsis, click on or hover over the header.

3. Select **Create IoT Edge Device**.

4. In the text box that opens, give your device an ID.

In the output screen, you see the result of the command. The device info is printed, which includes the **deviceId** that you provided and the **connectionString** that you can use to connect your physical device to your IoT hub.

### View all devices

All the devices that connect to your IoT hub are listed in the **Azure IoT Hub Devices** section of the Visual Studio Code Explorer. IoT Edge devices are distinguishable from non-Edge devices with a different icon, and the fact that they can be expanded to show the modules deployed to each device.

   ![View all IoT Edge devices in your IoT hub](./images/Register-device-vscode/view-devices.png)

### Retrieve the connection string

When you're ready to set up your device, you need the connection string that links your physical device with its identity in the IoT hub.

1. Right-click on the ID of your device in the **Azure IoT Hub Devices** section.

2. Select **Copy Device Connection String**.

   The connection string is copied to your clipboard.

You can also select **Get Device Info** from the right-click menu to see all the device info, including the connection string, in the output window.

## Step 2: Install IoT Edge on Windows Enterprise

### 1. Open a Powershell window as an Administrator

### 2. Run the **Deploy-IoTEdge** command,
 This command checks whether your Windows machine is on a supported version, turns on the containers feature, and then downloads the Moby runtime and the IoT Edge runtime. 

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Deploy-IoTEdge
```
Click Yes to the following pop-up questions. The machine will be restarted. After it's restarted, open a powershell window and run the command to finish the installation. 

### 3 Run the  Initialize-IoTEdge command to initialize the finish the IoT Edge installation

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Initialize-IoTEdge
```

### 4. Link to an IoT Edge device in IoTHub 

Provide the device connection string (saved from prior step). 

![IoTEdge-Installation](images/WinServer-Lab/IotEdge-Installation.png)

### 5. Run the Get-Service command to confirm IoT Edge runtime is installed and running

```powershell
Get-Service iotedge
```

![edge-running](images/WinServer-Lab/EdgeRunning.png)


### 6 : Confirm IoT Edge runtime is installed and running

You can check the Windows service is running: 

```powershell
Get-Service iotedge
```

To list all modules running in IoT Edge 
```powershell
iotedge list
```

## Step 3 : Download the sample file from repo, or git clone from the repository

[Squeeze Net Object Detection Sample](./SqueezeNetObjectDetectionSample.zip)

## Step 4 : Build Sample 

### 1. Check .Net installation 

Please make sure you have dotnet installed on your machine already. 

Run the following command to check

```powershell
dotnet --version
```

If not it's not installed, download and install dotnet 2.2 from: https://dotnet.microsoft.com/download/thank-you/dotnet-sdk-2.2.107-windows-x64-installer

### 2. git clone the sample repository or Unzip the downloaded file.

### 3. Build and publish the sample:


```powershell 
PS  C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs> dotnet publish -r win-x64

Microsoft (R) Build Engine version 15.8.169+g1ccb72aefa for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restore completed in 34.7 ms for C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs\SqueezeNetObjectDetection.csproj.
  SqueezeNetObjectDetection -> C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs\bin\Debug\netcoreapp2.1\win-x64\SqueezeNetObjectDetection.dll
  SqueezeNetObjectDetection -> C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs\bin\Debug\netcoreapp2.1\win-x64\publish\
```

## Step 5: Test the sample

As a first initial step, you can run the sample natively on your development machine to ensure it's working.

### 1. Run the app with the "--list" parameter to show the cameras on your PC:

```powershell
PS  C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs> dotnet run --list

Found 5 Cameras
Microsoft Camera Rear
Microsoft IR Camera Front
Microsoft Camera Front
icrosoftr LifeCam Studio(TM)
IntelIRCameraSensorGroup
```

### 2. Run the SqueezeNet model
From the camera list above, choose the camera to use as input to the --device parameter. In addition, pass along with the model using the --model parameter.

```powershell
PS  C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs> dotnet run --model=SqueezeNet.onnx --device=<camera name>

Loading modelfile 'SqueezeNet.onnx' on the 'default' device...
...OK 2484 ticks
Retrieving image from camera...
...OK 828 ticks
Running the model...
...OK 938 ticks
12/28/2018 12:13:05 PM Sending: {"results":[{"label":"coffee mug","confidence":0.960289478302002},{"label":"cup","confidence":0.035979188978672028},{"label":"water jug","confidence":6.35452670394443E-05}]}
```

Here we can see that the sample is successfully running on the development machine, found the camera, and recognized that the camera was probably
looking at a coffee mug. (It was.)

## Step 6 : Create Azure Container Registry

 A container repository is used to store container. In this lab, we're using the container registry to store the AI modules built from last steps. With the containers stored in Azure Container Registry, you can deploy the modules to the machines and devices where you want to run the moudule.

1. Select **Create a resource** > **Containers** > **Container Registry**.

**Registry name** enter a registry name
**Resource group** select the resource group used from prior lab 

use the default values for the other fields
 
2. Select **Create** to deploy.

![Creating a container registry in the Azure portal](images/IoTEnt-Lab/create-acr.png)

3. When the **Deployment succeeded** message appears, select the container registry in the portal. 

4. Select **Access keys** and **Enable** Admin user 

5. Take note of the value of the **Login server**, and copy one of the passwords. You'll need these values in the following steps.

![login server info](images/IoTEnt-Lab/acr-login-info.png)

## Step 7: Containerize the SqueezeNet app

In order for an app to be run as an edge module, the app needs to be containerize first. In the section we're going to walk through the steps to containerize an app. 

### 1. Set up Docker Environment 

1. Set the **WINDOW_WINMD** environment variable

To get access to Windows.AI.MachineLearning and various other Windows classes an assembly reference needs to be added for Windows.winmd

The file path for the Windows.winmd file may be: 

```powershell
setx /m WINDOWS_WINMD "C:\Program Files (x86)\Windows Kits\10\UnionMetadata\[version]\Windows.winmd"
```


2. Uninstall Docker CE

- check Docker versions: 
```powershell 
PS C:\WINDOWS\system32> docker version
Client: Docker Engine - Community
 Version:           18.09.2
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        6247962
 Built:             Sun Feb 10 04:12:31 2019
 OS/Arch:           windows/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.2
  API version:      1.39 (minimum version 1.24)
  Go version:       go1.10.6
  Git commit:       6247962
  Built:            Sun Feb 10 04:28:48 2019
  OS/Arch:          windows/amd64
  Experimental:     false
  ```
- if you see Server: Docker Engine - Community. 

Type docker at the Windows search bar at the low left of the screen -> Docker Desktop App -> Uninstall 

![uninstall docker ce](images/IoTEnt-Lab/docker-ce-uninstall.png)

3. Download Docker Client from: 

[Docker's Master Binaries Repo](https://master.dockerproject.org/windows/x86_64/docker.exe)

Save the downloaded docker.exe to the home directory of the home directory of the squeezenet app, for instance: C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>


4. Verify docker version

Run the following command to the versions of docker on your machine. Make sure The server should be version 3.0.3 or higher, built Jan 24 2019 or later. The client should be built Feb 12 2019 or later.

```powershell

.\Docker.exe version

```
![docker version](images/IotEnt-Lab/docker-version.png)

### 2. Set DOCKER_HOST environment variable 

```powershell
setx /m  DOCKER_HOST npipe:////./pipe/iotedge_moby_engine
```

### 3. Create docker image for the SqueezeNet app

<!-- Switch docker container: click the up arrow on the low right corner of the your screen -> right click on the docker icon -> select Switch to Windows Containers... 

it takes a minute or two, once the switch is done, you will docker sub menu something like the following

![switch container](images/IotEnt-Lab/switch-container.png) -->

```powershell
PS C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs> docker build .\bin\Debug\netcoreapp2.2\win-x64\publish\ -t {ACR_NAME}.azurecr.io/squeezenet

Sending build context to Docker daemon  84.56MB
Step 1/5 : FROM mcr.microsoft.com/windows:1809
1809: Pulling from windows
e7f2973ed2dd: Pull complete
a199e0ac5ac4: Pull complete
Digest: sha256:72bc5e6160d46116f3a31a452ada2b7592b3ca1e25e75fb2419ccf125c141798
Status: Downloaded newer image for mcr.microsoft.com/windows:1809
 ---> a03d8585d5d3
Step 2/5 : ARG EXE_DIR=.
 ---> Running in b5d9d49d9149
Removing intermediate container b5d9d49d9149
 ---> e0df5f439765
Step 3/5 : WORKDIR /app
 ---> Running in 14934127fa07
Removing intermediate container 14934127fa07
 ---> ca73548a239d
Step 4/5 : COPY $EXE_DIR/ ./
 ---> aeea90a5a119
Step 5/5 : CMD [ "SqueezeNetObjectDetection.exe", "-mSqueezeNet.onnx", "-dLifeCam", "-ef" ]
 ---> Running in f94a8f3bd485
Removing intermediate container f94a8f3bd485
 ---> f0e554508d5a
Successfully built f0e554508d5a
Successfully tagged RLACR.azureacr.io/squeezenet:1.0.0-x64
PS C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>
PS C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>
```

### 4. Test the object detection app in the container

One more test to ensure that the app is able to see the camera through the container.

```powershell

PS C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs> docker run --isolation process --device "/class/E5323777-F976-4f5b-9B55-B94699C46E44" {ACR_NAME}.azurecr.io/squeezenet SqueezeNetObjectDetection.exe --list

```

Note: The class type GUID is for the Microsoft LifeCam Cinema Cameras. 

## Step 8: Push the container to Azure Container Registry

### 1. Login to your Azure Container Registry

```
PS C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs> docker login {ACR_NAME}.azurecr.io
Username: {ACR_NAME}
Password:
Login Succeeded
```
### 2. Push the Docker image to AZure Container Registry

```powershell 
c:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>docker tag f0e554508d5a rlacr.azurecr.io/squeezenet

c:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>docker push rlacr.azurecr.io/squeezenet
The push refers to repository [rlacr.azurecr.io/squeezenet]
35d1c3903b3e: Pushed
b57f5f3fca07: Pushed
d5c14195a7b2: Pushed
60f1f6925582: Pushed
36fea90a67a6: Skipped foreign layer
bf16baf8d810: Skipped foreign layer
latest: digest: sha256:cb8921b98a999e2fbba4029b9fc1c266b42cce9b60eb3e4d8fb290595e2ab5ab size: 1783

```

### Step 9: Deploy module to IoT Edge 

### 1: Edit the deployment.json file

This is the deployment manifest file where you can specific modules to be deployed, along with different parameters and configuration info.  

Open the deployment.win-x64.json file. 

```powershell

C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>code deployment.win-x64.json

```

Fill in values to your container registry. Make sure the {ACR_IMAGE} name is the exact same image name as pushed to ACR last step. 

```
    "$edgeAgent": {
      "properties.desired": {
        "runtime": {
          "settings": {
            "registryCredentials": {
              "{ACR_NAME}": {
                "username": "{ACR_USER}",
                "password": "{ACR_PASSWORD}",
                "address": "{ACR_NAME}.azurecr.io"
              }
            }
          }
        }
...
        "modules": {
            "squeezenet": {
            "settings": {
              "image": "{ACR_IMAGE}",
              "createOptions": "{\"HostConfig\":{\"Devices\":[{\"CgroupPermissions\":\"\",\"PathInContainer\":\"\",\"PathOnHost\":\"class/E5323777-F976-4f5b-9B55-B94699C46E44\"},{\"CgroupPermissions\":\"\",\"PathInContainer\":\"\",\"PathOnHost\":\"class/5B45201D-F2F2-4F3B-85BB-30FF1F953599\"}],\"Isolation\":\"Process\"}}"
            }
          }
```

### 2: Deploy the SqueezeNet edge modules to IoT Edge

For production systems, after an edge module is developed and tested, it will be deployed edge device(s), which would be different machines than the development machine. In this lab, the edge device and the development machine are the same, so we'll be deploying this module to the lab machine which also serves as an IoT Edge device.

1. In the Visual Studio Code explorer view, expand the **Azure IoT Hub Devices** section.

2. Right-click on the IoT Edge device that you want to configure with the deployment manifest.

3. Select **Create Deployment for Single Device**.

4. Navigate to the deployment manifest JSON file (i.e C:\IoT-AI-Sample\Samples\EdgeModules\SqueezeNetObjectDetection\cs>code deployment.win-x64.json), and click **Select Edge Deployment Manifest**.

The results of your deployment are printed in the VS Code output. Successful deployments are applied within a few minutes if the target device is running and connected to the internet.

## Step 10: Confirm the SqueezeNet module has been deployed

Notice that the squeezenet module has been added to the edge device

![squeezenet on port](images/IoTEnt-Lab/squeezenet-running-portal.png)

You can also check the module running on your local machine from CLI

![squeezenet on cli](images/IoTEnt-Lab/squeezenet-running-cli.png)

To the view logs of the squeezenet module
```powershell
iotege logs <name of the moduel>
```

![squeezenet logs](images/IoTEnt-Lab/squeezenet-logs.png)

### Step 11: Stop IOT Edge 

To  stop IoT Edge, you can run the command below: 

```powershell
Stop-Service iotedge
```

Done! 