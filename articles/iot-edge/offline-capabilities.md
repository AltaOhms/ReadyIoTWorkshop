---
title: Operate devices offline - Azure IoT Edge | Microsoft Docs 
description: Understand how IoT Edge devices and modules can operate without internet connection for extended periods of time, and how IoT Edge can enable regular IoT devices to operate offline too.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 01/30/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
---

# Understand extended offline capabilities for IoT Edge devices, modules, and child devices (preview)

Azure IoT Edge supports extended offline operations on your IoT Edge devices, and enables offline operations on non-Edge child devices too. As long as an IoT Edge device has had one opportunity to connect to IoT Hub, it and any child devices can continue to function with intermittent or no internet connection. 

>[!NOTE]
>Offline support for IoT Edge is in [public preview](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## How it works

When an IoT Edge device goes into offline mode, the IoT Edge hub takes on three roles. First, it stores any messages that would go upstream and saves them until the device reconnects. Second, it acts on behalf of IoT Hub to authenticate modules and child devices so that they can continue to operate. Third, it enables communication between child devices that normally would go through IoT Hub. 

The following example shows how an IoT Edge scenario operates in offline mode:

1. **Configure an IoT Edge device.**

   IoT Edge devices automatically have offline capabilities enabled. To extend that capability to other IoT devices, you need to declare a parent-child relationship between the devices in IoT Hub. 

2. **Sync with IoT Hub.**

   At least once after installation of the IoT Edge runtime, the IoT Edge device needs to be online to sync with IoT Hub. In this sync, the IoT Edge device gets details about any child devices assigned to it. The IoT Edge device also securely updates its local cache to enable offline operations and retrieves settings for local storage of telemetry messages. 

3. **Go offline.**

   While disconnected from IoT Hub, the IoT Edge device, its deployed modules, and any children IoT devices can operate indefinitely. Modules and child devices can start and restart by authenticating with the IoT Edge hub while offline. Telemetry bound upstream to IoT Hub is stored locally. Communication between modules or between child IoT devices is maintained through direct methods or messages. 

4. **Reconnect and resync with IoT Hub.**

   Once the connection with IoT Hub is restored, the IoT Edge device syncs again. Locally stored messages are delivered in the same order in which they were stored. Any differences between the desired and reported properties of the modules and devices are reconciled. The IoT Edge device updates any changes to its set of assigned child IoT devices.

## Restrictions and limits

The extended offline capabilities described in this article are available in [IoT Edge version 1.0.4 or higher](https://github.com/Azure/azure-iotedge/releases). Earlier versions have a subset of offline features. Existing IoT Edge devices that don't have extended offline capabilities can't be upgraded by changing the runtime version, but must be reconfigured with a new IoT Edge device identity to gain these features. 

Extended offline support is available in all regions where IoT Hub is available, **except** East US.

Only non-Edge IoT devices can be added as child devices. 

IoT Edge devices and their assigned child devices can function indefinitely offline after the initial, one-time sync. However, storage of messages depends on the time to live (TTL) setting and the available disk space for storing the messages. 

## Set up an IoT Edge device

For an IoT Edge device to extend its extended offline capabilities to child IoT devices, you need to declare the parent-child relationships in the Azure portal.

### Assign child devices

Child devices can be any non-Edge device registered to the same IoT Hub. You can manage the parent-child relationship on creating a new device, or from the device details page of either the parent IoT Edge device or the child IoT device. 

   ![Manage child devices from the IoT Edge device details page](./media/offline-capabilities/manage-child-devices.png)

Parent devices can have multiple child devices, but a child device can only have one parent.

### Specifying DNS servers 

To improve robustness, it is recommended you specify the DNS server addresses used in your environment. For example, on Linux, update **/etc/docker/daemon.json** (you might need to create the file) to include:

```json
{
    "dns": ["1.1.1.1"]
}
```

If you are using a local DNS server, replace the 1.1.1.1 with the IP address of the local DNS server. Restart the docker service for the changes to take effect.


## Optional offline settings

If you expect to collect all the messages that your devices generate during long offline periods, configure the IoT Edge hub so that it can store all the messages. There are two changes that you can make to IoT Edge hub to enable long-term message storage. First, increase the time to live setting. Then, add additional disk space for message storage. 

### Time to live

The time to live setting is the amount of time (in seconds) that a message can wait to be delivered before it expires. The default is 7200 seconds (two hours). 

This setting is a desired property of the IoT Edge hub, which is stored in the module twin. You can configure it in the Azure portal, in the **Configure advanced Edge Runtime settings** section, or directly in the deployment manifest. 

```json
"$edgeHub": {
    "properties.desired": {
        "schemaVersion": "1.0",
        "routes": {},
        "storeAndForwardConfiguration": {
            "timeToLiveSecs": 7200
        }
    }
}
```

### Additional offline storage

Messages are stored by default in the IoT Edge hub's container filesystem. If that amount of storage isn't sufficient for your offline needs, you can dedicate local storage on the IoT Edge device. Create an environment variable for the IoT Edge hub that points to a storage folder in the container. Then, use the create options to bind that storage folder to a folder on the host machine. 

You can configure environment variables and the create options for the IoT Edge hub module in the Azure portal in the **Configure advanced Edge Runtime settings** section. Or, you can configure it directly in the deployment manifest. 

```json
"edgeHub": {
    "type": "docker",
    "settings": {
        "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
        "createOptions": {
            "HostConfig": {
                "Binds": ["<HostStoragePath>:<ModuleStoragePath>"],
                "PortBindings": {
                    "8883/tcp": [{"HostPort":"8883"}],
                    "443/tcp": [{"HostPort":"443"}],
                    "5671/tcp": [{"HostPort":"5671"}]
                }
            }
        }
    },
    "env": {
        "storageFolder": {
            "value": "<ModuleStoragePath>"
        }
    },
    "status": "running",
    "restartPolicy": "always"
}
```

Replace `<HostStoragePath>` and `<ModuleStoragePath>` with your host and module storage path; both host and module storage path must be an absolute path. In the create options, bind the host and module storage paths together. Then, create an environment variable that points to the module storage path.  

For example, `"Binds":["/etc/iotedge/storage/:/iotedge/storage/"]` means the directory **/etc/iotedge/storage** on your host system is mapped to the directory **/iotedge/storage/** on the container. Or another example for Windows systems, `"Binds":["C:\\temp:C:\\contemp"]` means the directory **C:\\temp** on your host system is mapped to the directory **C:\\contemp** on the container. 

You can also find more details about create options from [docker docs](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate).

## Next steps

Enable extended offline operations in your [transparent gateway](how-to-create-transparent-gateway.md) scenarios.
