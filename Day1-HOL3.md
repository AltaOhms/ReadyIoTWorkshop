# Windows 10 IoT Core + Azure IoT Edge (60 Min)

Many IoT solutions use analytics services to gain insight about data as it arrives in the cloud from the IoT devices. With Azure IoT Edge, you can take [Azure Stream Analytics](https://docs.microsoft.com/azure/stream-analytics/) logic and move it onto the device itself. By processing telemetry streams at the edge, you can reduce the amount of uploaded data and reduce the time it takes to react to actionable insights.

Azure IoT Edge and Azure Stream Analytics are integrated so that you can create an Azure Stream Analytics job in the Azure portal and then deploy it as an IoT Edge module with no additional code.  

Azure Stream Analytics provides a richly structured query syntax for data analysis both in the cloud and on IoT Edge devices. For more information about Azure Stream Analytics on IoT Edge, see [Azure Stream Analytics documentation](../stream-analytics/stream-analytics-edge.md).

The Stream Analytics module in this lab calculates the average temperature over a rolling 30-second window. When that average reaches 70, the module sends an alert for the device to take action. In this case, that action is to reset the simulated temperature sensor. In a production environment, you might use this functionality to shut off a machine or take preventative measures when the temperature reaches dangerous levels.

In this lab, you learn how to:

- How to manipulate Azure Resources from Azure Command Line Interface (AZ CLI)
- Create an Azure Stream Analytics job to process data on the edge.
- Connect Azure Stream Analytics job with other IoT Edge modules.
- Deploy the Azure Stream Analytics job to an Azure IoT Core device.

Windows IoT Core device is a small device with lesser resources, so we'll be doing most the steps on your lab development machine, then deploy the module to the IoT Core device.

## Prerequisites

- An instance of IoT Hub from the previous HOL
- IoT Core device, for instance an Intel Compute Stick PC
- AZ CLI and IoT Extension  
    They are preinstalled into your HOL Windows 10 Dev Machine

References :

- AZ CLI : [https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest)
- IoT Extension : [https://github.com/Azure/azure-iot-cli-extension](https://github.com/Azure/azure-iot-cli-extension)

## Step 1 : Register the IoT Core device as an Azure IoT Edge device (On your DevEnv)

Command line interface (CLI) is the only interface available in some occasions, so in this lab, we are going to use the Azure Command Line Interface (AZ CLI) to create devices.

### Step 1.1 : Sign in to your Azure Account

1. Open a command prompt, Powershell prompt, or VSCode Terminal
1. Run `az login` command which will open a web browser, and sign in to your account with your credential

```bash
az login
```

### Step 1.2 : Create a IoT Edge device

Use the following command to create a new device identity in your IoT hub:

```bash
az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled
```

This command includes three parameters:

|Parameters  |Description  |Example  |
|---------|---------|---------|
|Device ID | Provide a descriptive name of your IoT Core device, which is unique to your IoT hub.         | MyIoTCore-01        |
|Hub Name     | Provide the name of your IoT hub to which your IoT Edge device will connect  |MsIoTBootCamp1234        |
|Edge Enabled     |This parameter declares that the device is for use with IoT Edge         |         |

![az iot hub device-identity create output](images/IotCore-Lab/Create-edge-device.png)

> [!TIP]  
> If your environment did not have the device-identity extension installed, run the following command  
>  
>  ```bash  
>   az extension add --name azure-cli-iot-ext  
>```

### Step 1.3 : Check the new device

Run the following command to view all devices in your IoT hub:

```bash
az iot hub device-identity list --hub-name [hub name]
```

Example :

> [!TIP]  
> You should see 3 devices at this point

```ps
PS C:\> az iot hub device-identity list --hub-name MsIoTBootCamp1234
[
  {
    "authenticationType": "sas",
    "capabilities": {
      "iotEdge": true
    },
    :
    <Snip>
    :
  },
  {
    "authenticationType": "sas",
    "capabilities": {
      "iotEdge": true   <<<=====================
    },
    "cloudToDeviceMessageCount": 0,
    "connectionState": "Disconnected",
    "deviceEtag": "NzE3NTI2NTQ0",
    "deviceId": "MyIoTCore-01",
    "deviceScope": "ms-azure-iot-edge://MyIoTCore-01-636956957779426343",
    "etag": "AAAAAAAAAAE=",
    "lastActivityTime": "0001-01-01T00:00:00Z",
    "properties": {
      "desired": {
        "$metadata": {
          "$lastUpdated": "2019-06-09T16:49:37.9426343Z"
        },
        "$version": 1
      },
      "reported": {
        "$metadata": {
          "$lastUpdated": "2019-06-09T16:49:37.9426343Z"
        },
        "$version": 1
      }
    },
    "status": "enabled",
    "statusUpdateTime": "0001-01-01T00:00:00Z",
    "tags": {
      "$metadata": {
        "$lastUpdated": null
      },
      "$version": 1
    },
    "version": 2,
    "x509Thumbprint": {
      "primaryThumbprint": null,
      "secondaryThumbprint": null
    }
  },
  {
    :
    <Snip>
    :
  }
]

```

> [!TIP]  
> Any device that is registered as an IoT Edge device will have the property  
> **capabilities.iotEdge** set to **true**  
>  
> ```json  
>     "capabilities": {
>      "iotEdge": true
>    },
> ```

### Step 1.4 : Retrieve the connection string

When you're ready to set up your device, you need the connection string that links your physical device with its identity in the IoT hub. Use the following command to return the connection string for a single device:

```bash
az iot hub device-identity show-connection-string --device-id [device id] --hub-name [hub name]
```

> [!TIP]  
> The value for the `device-id` parameter is case-sensitive. Don't copy the quotation marks around the connection string

Example :

```ps
PS C:\> az iot hub device-identity show-connection-string --device-id MyIoTCore-01 --hub-name MsIoTBootCamp1234
{
  "connectionString": "HostName=MsIoTBootCamp1234.azure-devices.net;DeviceId=MyIoTCore-01;SharedAccessKey=&()*(Fdafdkafda90(*)(*("
}
```

## Step 2 : IoT Core Dashboard

IoT Core Dashboard is a tool to help you download, connect, and configure your Windows 10 IoT Core device.  
Using IoT Core Dashboard, we will  

- Identify your Windows 10 IoT Core Device
- Connect to your Windows 10 IoT Core Device

### Step 2.1 : Launch IoT Core Dashboard

1. Install IoT Core Dashboard  

    > [!TIP]  
    > IoT Core Dashboard setup file is already downloaded on your desktop  
    > ![IoTCoreDashboardIcon](images/IoTCore-Lab/IoTCoreDashboardIcon.PNG)
  
    If you do not have installation file, download and install IoT Core Dashboard from: [https://docs.microsoft.com/en-us/windows/iot-core/connect-your-device/iotdashboard](https://docs.microsoft.com/en-us/windows/iot-core/connect-your-device/iotdashboard)

2. Launch IoT Core Dashboard
3. Open **My devices** tab  
    If there is any Windows 10 IoT Core devices on the same network, they will be listed here

![MyDevices](images/IoTCore-Lab/Dashboard-MyDevice.png)

## Step 3: Boot Windows 10 IoT Core

The Intel Compute Stick is the IoT Core device for this lab.

> [!NOTE]  
> Please pick up your Windows 10 IoT Core from instructor  
> The kit is consist of followings  
> Kit #1 (With 10 inch HDMI Monitor)
>
> - Intel Compute Stick + AC Adapter
> - HDMI Monitor + AC Adapter
> - HDMI Extension Cable
>
> Kit #2 (With 15" HDMI Monitor)
>
> - Intel Compute Stick + AC Adapter
> - HDMI Monitor + AC Adapter + USB Cable
> - HDMI cable + Female-Female Adapter  

### Step 3.1 : Connect to your IoT Core device

1. Launch IoT Core Dashboard
1. Open `My device` tab and locate your IoT Core device

  > [!TIP]  
  > Please check IP Address of Windows 10 IoT Core device to locate your device

1. Right click on the device and select **Launch Powershell**  
    Use following credential to login to Windows 10 IoT Core

    > [!NOTE]  
    > IoT Core Credential  
    > User Name : localhost\administrator  
    > Password  : bootcamp

    > [!TIP]  
    > It may take a minutes or so during the first remote Powershell connection  
    >  
    > Please ensure you are in Powershell Remote Console.  When you see IP Address in the beginning of command prompt, your are in Powershell Remote Console.  If not, please re-connect to your IoT Core device again.  
    >  
    > ![PSRemote](images/IoTCore-Lab/PSRemotePrompt.png)

  ![IoT Core Dashboard](images/IoTCore-Lab/IoTCoreDashboard.png)

### Step 3.2 : Deploy Azure IoT Edge Runtime

Run `Deploy-IoTEdge` cmdlet to deploy the runtime.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Deploy-IoTEdge -ContainerOs Windows
```

> [!NOTE]  
> This will require a reboot.  Re-connect to Powershell Remote Console upon reboot to continue this lab

### Step 3.3 : Initialize Azure IoT Edge Runtime

1. Run `Initialize-IoTEdge` cmdlet to complete the installation process

    ```powershell
    . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Initialize-IoTEdge -ContainerOs Windows
    ```

1. Provide the device connection string from [above](#step-14--retrieve-the-connection-string)  
  
    Output Example:
  
    ![IoTEdgeInstall](images/IoTCore-Lab/IoTEdgeInstall.png)

### Step 3.4 : Verify IoT Edge Runtime initialization

Confirm IoT Edge runtime is installed and running on the IoT Core device.  Initialization may take a few minutes.  

- Check the status of the IoT Edge service with `Get-Service` cmdlet
  
    ```powershell
    Get-Service iotedge
    ```  

- Check running module with `iotedge list` command

    ```ps
    iotedge list
    ```

    > [!TIP]  
    > Only edgeAgent is expected to be up and running at this point  
    >
    > ```ps  
    > [10.123.98.82]: PS C:\Data\Users\Administrator\Documents> iotedge list
    > NAME             STATUS           DESCRIPTION      CONFIG  
    > edgeAgent        running          Up 9 hours       mcr.microsoft.com/azureiotedge-agent:1.0.7.1  
    > ```

- Check logs from the runtime with `Get-IoTEdgelog` cmdlet

    ```ps  
    Get-IoTEdgeLog
    ```

## Step 4 : Deploy Simulated Temperature Sensor

Deploy Simulated Temperature Sensor module to Windows 10 IoT Core device with one of following options :

- From Azure Marketplace in Azure Portal (Instruction [Here](Day1-HOL1.md#step-6--deploy-simulated-temperature-sensor-from-marketplace-to-windows-server))
- Simulated Temperature Sensor module you built and pushed to your ACR (Instruction [Here](Day1-HOL2.md#step-8--deploy-to-your-windows-10))

Once deployed, please make sure you can see messages from Simulated Temperature Sensor

- Logs from the module `iotedge logs -f SimulatedTemperatureSensor` command (Instruction [Here](Day1-HOL1.md#step-72--confirm-messages-are-being-sent))
- With `Device Explorer` (Instruction [Here](Day1-HOL1.md#step-73--confirm-the-messages-are-received-by-iot-hub))

> [!TIP]  
> Complete og output from `edgeAgent` module during deployment, for your reference.

```ps
[10.123.98.82]: PS C:\Data\Users\Administrator\Documents> iotedge logs -f edgeAgent
[06/10/2019 12:30:34.180 PM] Edge Agent Main()
2019-06-10 05:30:36.529 -07:00 [INF] - Starting module management agent.
2019-06-10 05:30:38.400 -07:00 [INF] - Version - 1.0.7.1.22377503 (f7c51d92be8336bc6be042e1f1f2505ba01679f3)
2019-06-10 05:30:38.404 -07:00 [INF] -
        █████╗ ███████╗██╗   ██╗██████╗ ███████╗
       ██╔══██╗╚══███╔╝██║   ██║██╔══██╗██╔════╝
       ███████║  ███╔╝ ██║   ██║██████╔╝█████╗
       ██╔══██║ ███╔╝  ██║   ██║██╔══██╗██╔══╝
       ██║  ██║███████╗╚██████╔╝██║  ██║███████╗
       ╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝

 ██╗ ██████╗ ████████╗    ███████╗██████╗  ██████╗ ███████╗
 ██║██╔═══██╗╚══██╔══╝    ██╔════╝██╔══██╗██╔════╝ ██╔════╝
 ██║██║   ██║   ██║       █████╗  ██║  ██║██║  ███╗█████╗
 ██║██║   ██║   ██║       ██╔══╝  ██║  ██║██║   ██║██╔══╝
 ██║╚██████╔╝   ██║       ███████╗██████╔╝╚██████╔╝███████╗
 ╚═╝ ╚═════╝    ╚═╝       ╚══════╝╚═════╝  ╚═════╝ ╚══════╝

2019-06-10 05:30:39.783 -07:00 [INF] - Started operation refresh twin config
2019-06-10 05:30:39.978 -07:00 [INF] - Edge agent attempting to connect to IoT Hub via Amqp_Tcp_Only...
2019-06-10 05:30:45.375 -07:00 [INF] - Created persistent store at C:\Windows\TEMP\edgeAgent
2019-06-10 05:30:48.501 -07:00 [INF] - Edge agent connected to IoT Hub via Amqp_Tcp_Only.
2019-06-10 05:30:50.241 -07:00 [INF] - Obtained Edge agent twin from IoTHub with desired properties version 2 and reported properties version 2.
2019-06-10 05:30:53.279 -07:00 [INF] - Plan execution started for deployment 2
2019-06-10 05:30:53.478 -07:00 [INF] - Executing command: "Command Group: (
  [Create module edgeHub]
  [Start module edgeHub]
)"
2019-06-10 05:30:53.499 -07:00 [INF] - Executing command: "Create module edgeHub"
2019-06-10 05:31:21.087 -07:00 [INF] - Executing command: "Start module edgeHub"
2019-06-10 05:31:26.157 -07:00 [INF] - Executing command: "Command Group: (
  [Create module SimulatedTemperatureSensor]
  [Start module SimulatedTemperatureSensor]
)"
2019-06-10 05:31:26.157 -07:00 [INF] - Executing command: "Create module SimulatedTemperatureSensor"
2019-06-10 05:31:34.146 -07:00 [INF] - Executing command: "Start module SimulatedTemperatureSensor"
2019-06-10 05:31:38.275 -07:00 [INF] - Plan execution ended for deployment 2
2019-06-10 05:31:39.260 -07:00 [INF] - Updated reported properties
2019-06-10 05:31:44.674 -07:00 [INF] - Updated reported properties

```

## Azure Stream Analytics on IoT Edge

Azure Stream Analytics (ASA) is a fully managed, serverless event processing engine that allows you to analyze data streams from applications and IoT devices.  

ASA on IoT Edge enables intelligence closer to IoT Devices for near real time analysis with low latency, resiliency, efficient use of bandwidth, and compliance.

In this section, you will create an ASA on IoT Edge job, and deploy to your IoT Edge device.

- Create a storage container
- Create an ASA edge job
- Deploy ASA on your Windows 10 IoT Core device

Reference :

- [https://docs.microsoft.com/en-us/azure/stream-analytics/](https://docs.microsoft.com/en-us/azure/stream-analytics/)
- [https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-edge](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-edge)

## Step 5 : Create a storage account

ASA edge job stores compiled query and job configuration into storage (blob).  
In this step, we will create a new storage account for your ASA Edge Job.

1. In the Azure portal, go to **Create a resource** > **Storage** > **Storage account**  
  
    ![StorageAccount1](images/IoTCore-Lab/StorageAccount1.png)

1. Provide the following information to create your storage account  
    Keep all default options for ASA edge job  
    See table below for description and example field data

1. Click on **Review + create**, then **Create** on the next page to start deployment

    ![StorageAccount2](images/IoTCore-Lab/StorageAccount2.png)

|Field    |Description |Example  |
|---------|---------|---------|
|Subscription         | Your Subscription        |         |
|Resource Group       | Resource Group you created from previous labs | IoTBootCamp2019        |
|Storage Account Name | Only lowercase letters and numbers            | bootcamphol3stor        |
|Location             | Location of data center.  Use the same region as your IoT Hub       | West US        |

## Step 6 : Create a new ASA Edge job

ASA job is consist of a few parts

- ASA job itself
- Job definition  
    ASA job takes data (input), runs analytics (query), then generates results (output)
    ![asa0](images/IoTCore-Lab/ASA0.png)

1. In Azure Portal, create a new resource or click [here](https://ms.portal.azure.com/#create/Microsoft.StreamAnalyticsJob)

    ![asa1](images/IoTCore-Lab/ASA1.png)

1. Provide the following information to create a new ASA Edge job  
    See table below for description and example field data

    ![asa2](images/IoTCore-Lab/ASA2.png)

1. Click **Create** and wait for deployment to complete

|Field    |Description |Example  |
|---------|---------|---------|
|Job Name | A name for your job        | MyASAonEdgeJob         |
|Subscription         | Subscription used for this lab                |                  |
|Resource Group       | Resource Group you created from previous labs | IoTBootCamp2019        |
|Location             | Location of data center.  Use the same region as your IoT Hub       | West US        |
|Hosting environment  | Determines the job runs in Cloud or Edge.  Select **Edge** | Edge |

## Step 7 : ASA Edge Job Definition

Once your Stream Analytics job is created in the Azure portal, you can configure it with an input, an output, and a query to run on the data that passes through.

Using the three elements of input, output, and query, this section creates a job that :  

- Receives temperature data from the IoT Edge device as input
- Runs query to analyze that data in a rolling 30-second window
- Sends an alert to output if the average temperature in that window goes over 70 degrees
  
    ![asa3](images/IoTCore-Lab/ASA3.png)

### Step 7.1 : Add a stream input

Add a new **input** to receive messages from Simulated Temperature Sensor module

1. Navigate to your Stream Analytics job in the Azure portal

1. Under **Job Topology**, select **Inputs**

1. Click **Add stream input**

1. Choose **Edge Hub** from the drop-down list.

    ![asa4](images/IoTCore-Lab/ASA4.png)

1. In the **New input** pane, enter a unique name (e.g. temperature) as the input alias

1. Keep the default values for the other fields, and select **Save**

    ![asa5](images/IoTCore-Lab/ASA5.png)

1. Make sure the new input is created

    ![asa6](images/IoTCore-Lab/ASA6.png)

### Step 7.2 : Add a stream output

Add a new **output** to send alerts to Simulated Temperature Sensor module

1. Under **Job Topology**, open **Output**

1. Click **Add**

1. Choose **Edge Hub** from the drop-down list

    ![asa7](images/IoTCore-Lab/ASA7.png)

1. In the **New output** pane, enter a unique name (e.g. alert) as the output alias

1. Keep the default values for the other fields, and select **Save**.

    ![asa8](images/IoTCore-Lab/ASA8.png)

1. Make sure the new output is created

    ![asa9](images/IoTCore-Lab/ASA9.png)

### Step 7.3 : Add a query

Create a new query for analytics.  

The SQL code sends a reset command to the alert output if the average machine temperature in a 30-second window reaches 35 degrees. The reset command has been pre-programmed into the sensor as an action that can be taken.  You can define different timing windows with **Window functions**

1. Under **Job Topology**, select **Query**

1. Replace the default text with the following query

    ```sql
    SELECT  
        'reset' AS command
    INTO
       alert
    FROM
       temperature TIMESTAMP BY timeCreated
    GROUP BY TumblingWindow(second,30)
    HAVING Avg(machine.temperature) > 35
    ```

1. Select **Save**

![asa10](images/IoTCore-Lab/ASA10.png)

Reference :  

- Window functions : [https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions](https://docs.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions)

## Step 8 : Configure Storage for ASA Job

To prepare your Stream Analytics job to be deployed on an IoT Edge device, you need to associate the job with a container in a storage account. When you go to deploy your job, the job definition is exported to the storage container.

1. Under **Configure**, select **Storage account settings**

1. Select **Add storage account**

    ![asa11](images/IoTCore-Lab/ASA11.png)

1. Check **Select storage account from your subscription**

1. Select **Subscription**
    You should see storage account you [created](#step-5--create-a-storage-account)

1. Select storage account you created

1. Create a new **Container**

1. Give a unique name for the new container (e.g. myasajobcontainer)

1. Select **Save**

![asa12](images/IoTCore-Lab/ASA12.png)

## Step 9 : Deploy the Azure Stream Analytics job to your IoT Core device

You are now ready to deploy the Azure Stream Analytics job on your IoT Core device.

In this section, you use the **Set Modules** wizard in the Azure portal to create a *deployment manifest*. A deployment manifest is a JSON file that describes all the modules that will be deployed to a device, the container registries that store the module images, how the modules should be managed, and how the modules can communicate with each other. Your IoT Edge device retrieves its deployment manifest from IoT Hub, then uses the information in it to deploy and configure all of its assigned modules.

### Step 9.1 : Navigate to your IoT Core device Id page in Azure Portal

Navigate to your IoT Hub page in Azure Portal, then locate your IoT Edge device

![asa13](images/IoTCore-Lab/ASA13.png)

### Step 9.2 : Add ASA Edge job to your deployment

1. Select **Set modules**

1. Click **Add**

1. Select **Azure Stream Analytics Module**

1. Select your subscription and the Azure Stream Analytics Edge job that you created

1. Click **Save**

1. Click **Next**

![asa14](images/IoTCore-Lab/ASA14.png)

### Step 9.3 : Add Routes

IoT Edge modules send and receive data through Azure IoT Edge Runtime.  

Route rule defines how messages are sent and received.  In this step, we will add route rules so that :

- Messages from Simulated Temperature Sensor module are sent to cloud (IoT Hub)
- Messages from Simulated Temperature Sensor module are sent to ASA Edge module
- Alert from ASA Edge module to Simulated Temperature Sensor Module
- Alert from ASA Edge module to Cloud (IoT Hub)

In **Specify Routes** page, modify route rule

> [!IMPORTANT]  
>
> - Replace **tempSensor** with your Simulated Temperature Sensor module name  
>   Check module name with `iotedge list` command in remote Powershell console  
> - Replace **AsaEdge** with your ASA Edge job name you [created](#step-6--create-a-new-asa-edge-job)

```json
{
    "routes": {
        "telemetryToCloud": "FROM /messages/modules/{tempSensor}/* INTO $upstream",
        "telemetryToAsa": "FROM /messages/modules/{tempSensor}/* INTO BrokeredEndpoint(\"/modules/{AsaEdge}/inputs/temperature\")",
        "alertsToReset": "FROM /messages/modules/{AsaEdge}/* INTO BrokeredEndpoint(\"/modules/{tempSensor}/inputs/control\")",
        "alertsToCloud": "FROM /messages/modules/{AsaEdge}/* INTO $upstream"
    }
}```

Example with **SimulatedTemperatureSensor** and **MyASAonEdgeJob**

```json
{
    "routes": {
        "telemetryToCloud": "FROM /messages/modules/SimulatedTemperatureSensor/* INTO $upstream",
        "telemetryToAsa": "FROM /messages/modules/SimulatedTemperatureSensor/* INTO BrokeredEndpoint(\"/modules/MyASAonEdgeJob/inputs/temperature\")",
        "alertsToReset": "FROM /messages/modules/MyASAonEdgeJob/* INTO BrokeredEndpoint(\"/modules/SimulatedTemperatureSensor/inputs/control\")",
        "alertsToCloud": "FROM /messages/modules/MyASAonEdgeJob/* INTO $upstream"
    }
}
```

![asa15](images/IoTCore-Lab/ASA15.png)

### Step 9.4 : Check deployment

Monitor and check deployment progress and results with following commands

- `iotedge log edgeAgent`  
    You should see logs like this example

    ```bash
    [10.123.98.82]: PS C:\Data\Users\Administrator\Documents> iotedge list
        :
      <Snip>
        :
    2019-06-10 07:45:04.013 -07:00 [INF] - Executing command: "Create module MyASAonEdgeJob"
    2019-06-10 07:45:35.432 -07:00 [INF] - Executing command: "Start module MyASAonEdgeJob"
    2019-06-10 07:45:40.678 -07:00 [INF] - Plan execution ended for deployment 3
    2019-06-10 07:45:40.882 -07:00 [INF] - Updated reported properties
    2019-06-10 07:45:46.162 -07:00 [INF] - Updated reported properties
    ```

- `iotedge list`  
    You should see 4 modules in running status  
  
    ```bash
    [10.123.98.82]: PS C:\Data\Users\Administrator\Documents> iotedge list
    NAME                        STATUS           DESCRIPTION      CONFIG
    MyASAonEdgeJob              running          Up 3 minutes     mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.2
    SimulatedTemperatureSensor  running          Up 2 hours       mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0
    edgeHub                     running          Up 2 hours       mcr.microsoft.com/azureiotedge-hub:1.0
    edgeAgent                   running          Up 2 hours       mcr.microsoft.com/azureiotedge-agent:1.0
    ```

- `iotedge logs <Your ASA on Edge Job Module Name>`  
    You should see logs like this example

    ```bash
    [10.123.98.82]: PS C:\Data\Users\Administrator\Documents> iotedge logs MyASAonEdgeJob
    2019-06-10 07:45:46.685 -07:00 [INF] - Module client initializing ...
    2019-06-10 07:45:58.191 -07:00 [INF] - Opened module client connection
    2019-06-10 07:46:01.098 -07:00 [INF] - ASA getting path: https://bootcamphol3stor.blob.core.windows.net/myasajobcontainer/ASAEdgeJobs/e3d2efb5-acca-484c-abe1-ff2ad9e14076/40fb1b37-55bc-4ca5-b403-554d57410901/ASAEdgeJobDefinition.zip?sv=2018-03-28&sr=b&sig=RogFBCpCzyZ4cgInlq8xTM3cii8EVCwiygdcieU7eEI%3D&st=2019-06-10T14%3A21%3A35Z&se=2022-06-10T14%3A31%3A35Z&sp=r
    2019-06-10 07:46:04.285 -07:00 [INF] - Unzip ASA job definition...
    2019-06-10 07:46:04.531 -07:00 [INF] - Initializing ASA Engine ...
    2019-06-10 07:46:11.163 -07:00 [INF] -
    =============================Metrics report=============================
     Errors                                       :                   0
     InputEvents                                  :                   0
     DeserializationError                         :                   0
     OutputWatermarkDelaySeconds                  :                 N/A
     InputEventBytes                              :                 0 B
     InputEventsSources                           :                   0
     EarlyInputEvents                             :                   0
     ConversionErrors                             :                   0
     LateInputEvents                              :                   0
     OutputEvents                                 :                   0
     DroppedOrAdjustedEvents                      :                   0
    ========================================================================
  
    2019-06-10 07:46:38.166 -07:00 [INF] - ASA module registered message endpoint : [temperature]
    2019-06-10 07:46:47.252 -07:00 [INF] - Module reported job status successfully, Job Status : Idle, Is Runtime Error : False
    ```

> [!NOTE]  
> Simulated Temperature Sensor module stops sending messages after 500 messages.  
> If you see logs saying **Done sending 500 massages**, restart the module to start sending another 500 messages.  
>  
> To restart SimulatedTemperatureSensor module, run `iotedge restart <your module name>`  
> E.g. `iotedge restart SimulatedTemperatureSensor`  
>
```ps
iotedge logs -f SimulatedTemperatureSensor
   :
 <Snip>
   :
6/10/2019 6:13:58 AM> Sending message: 495, Body: [{"machine":{"temperature":100.64361063735272,"pressure":10.073322730837651},"ambient":{"temperature":20.681532002138688,"humidity":24},"timeCreated":"2019-06-10T13:13:58.684986Z"}]  
Done sending 500 messages
```

## Step 10 : Check ASA Query Result

The query we created for ASA Edge Job sends **reset** command when a 30-second window reaches 35 degrees.  We also defined Route Rule to send message to Cloud (IoT Hub)

Confirm this with logs from :  

- Simulated Temperature Sensor log  
- Device Explorer  

Examples :

- `iotedge logs -f SimulatedTemperatureSensor`  
  
```cmd
        6/11/2019 12:50:19 AM> Sending message: 67, Body: [{"machine":{"temperature":36.115638783139005,"pressure":2.7220347980791271},"ambient":{"temperature":21.238499270630303,"humidity":24},"timeCreated":"2019-06-11T07:50:19.8207774Z"}]
        6/11/2019 12:50:24 AM> Sending message: 68, Body: [{"machine":{"temperature":36.766889925239084,"pressure":2.796227966166478},"ambient":{"temperature":20.82198495479393,"humidity":25},"timeCreated":"2019-06-11T07:50:24.962392Z"}]
        6/11/2019 12:50:30 AM> Sending message: 69, Body: [{"machine":{"temperature":37.231523926826895,"pressure":2.84916095368914},"ambient":{"temperature":21.469507630900249,"humidity":26},"timeCreated":"2019-06-11T07:50:30.0242983Z"}]
                                                                        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Received message Body: [{"command":"reset"}]   <<===========================
Resetting temperature sensor..
        6/11/2019 12:50:35 AM> Sending message: 70, Body: [{"machine":{"temperature":21.815419431573442,"pressure":1.0928958846096326},"ambient":{"temperature":20.773474650119187,"humidity":25},"timeCreated":"2019-06-11T07:50:35.1582115Z"}]
                                                                        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

        6/11/2019 12:50:40 AM> Sending message: 71, Body: [{"machine":{"temperature":22.16299533083243,"pressure":1.1324931389555934},"ambient":{"temperature":20.758661693082498,"humidity":24},"timeCreated":"2019-06-11T07:50:40.2224446Z"}]
```
  
- Device Explorer

![ASA16](images/IoTCore-Lab/ASA16.png)

## Finished

You have successfully deployed ASA Edge job to your Windows 10 IoT Core device.

## Optional Challenger lab

Add Azure Function module to filter data from Simulated Temperature Sensor

### Key Points

- These samples contains dockerfile for Windows and Linux  
    Please make sure you use Windows Container
- Module compile/build steps are very similar to what you have done in [Lab 2](Day1-HOL2.md)
- Use one of two samples modules as a example

Reference :

- [https://azure.microsoft.com/en-us/resources/samples/azure-functions-iot-timeseries-analytics/](https://azure.microsoft.com/en-us/resources/samples/azure-functions-iot-timeseries-analytics/)
- [https://github.com/Azure/iotedge/tree/master/edge-modules/TemperatureFilter](https://github.com/Azure/iotedge/tree/master/edge-modules/TemperatureFilter)

Hint :

- [https://github.com/Azure/iotedge/tree/master/edge-modules/functions](https://github.com/Azure/iotedge/tree/master/edge-modules/functions)
