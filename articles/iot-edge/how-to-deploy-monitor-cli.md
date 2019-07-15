---
title: Create automatic deployments from command line - Azure IoT Edge | Microsoft Docs 
description: Use the IoT extension for Azure CLI to create automatic deployments for groups of IoT Edge devices
keywords: 
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 02/19/2019
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
---

# Deploy and monitor IoT Edge modules at scale using the Azure CLI

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-edge-how-to-deploy-monitor-selector.md)]

Azure IoT Edge enables you to move analytics to the edge, and provides a cloud interface so that you can manage and monitor your IoT Edge devices remotely. The capability to remotely manage devices is increasingly important as Internet of Things solutions are growing larger and more complex. Azure IoT Edge is designed to support your business goals, no matter how many devices you add.

You can manage individual devices and deploy modules to them one at a time. However, if you want to make changes to devices at a large scale, you can create an **IoT Edge automatic deployment**, which is part of Automatic Device Management in IoT Hub. Deployments are dynamic processes that enable you to deploy multiple modules to multiple devices at once, track the status and health of the modules, and make changes when necessary. 

In this article, you set up Azure CLI and the IoT extension. Then, you learn how to deploy modules to a set of IoT Edge devices and monitor the progress using the available CLI commands.

## CLI Prerequisites

* An [IoT hub](../iot-hub/iot-hub-create-using-cli.md) in your Azure subscription. 
* [IoT Edge devices](how-to-register-device-cli.md) with the IoT Edge runtime installed.
* [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli) in your environment. At a minimum, your Azure CLI version must be 2.0.24 or above. Use `az –-version` to validate. This version supports az extension commands and introduces the Knack command framework. 
* The [IoT extension for Azure CLI](https://github.com/Azure/azure-iot-cli-extension).

## Configure a deployment manifest

A deployment manifest is a JSON document that describes which modules to deploy, how data flows between the modules, and desired properties of the module twins. For more information about how deployment manifests work and how to create them, see [Understand how IoT Edge modules can be used, configured, and reused](module-composition.md).

To deploy modules using Azure CLI, save the deployment manifest locally as a .txt file. You will use the file path in the next section when you run the command to apply the configuration to your device. 

Here's a basic deployment manifest with one module as an example:

   ```json
   {
        "content": {
         "modulesContent": {
           "$edgeAgent": {
             "properties.desired": {
               "schemaVersion": "1.0",
               "runtime": {
                 "type": "docker",
                 "settings": {
                   "minDockerVersion": "v1.25",
                   "loggingOptions": "",
                   "registryCredentials": {
                     "registryName": {
                       "username": "",
                       "password": "",
                       "address": ""
                     }
                   }
               },
               "systemModules": {
                 "edgeAgent": {
                   "type": "docker",
                   "settings": {
                     "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                     "createOptions": "{}"
                   }
                 },
                 "edgeHub": {
                   "type": "docker",
                   "status": "running",
                   "restartPolicy": "always",
                   "settings": {
                     "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                     "createOptions": "{}"
                   }
                 }
               },
               "modules": {
                 "tempSensor": {
                   "version": "1.0",
                   "type": "docker",
                   "status": "running",
                   "restartPolicy": "always",
                   "settings": {
                     "image": "mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0",
                     "createOptions": "{}"
                   }
                 }
               }
             }
           },
           "$edgeHub": {
             "properties.desired": {
               "schemaVersion": "1.0",
               "routes": {
                   "route": "FROM /* INTO $upstream"
               },
               "storeAndForwardConfiguration": {
                 "timeToLiveSecs": 7200
               }
             }
           },
           "tempSensor": {
             "properties.desired": {}
           }
         }
       }
   }
   ```

## Identify devices using tags

Before you can create a deployment, you have to be able to specify which devices you want to affect. Azure IoT Edge identifies devices using **tags** in the device twin. Each device can have multiple tags, and you can define them any way that makes sense for your solution. For example, if you manage a campus of smart buildings, you might add the following tags to a device:

```json
"tags":{
    "location":{
        "building": "20",
        "floor": "2"
    },
    "roomtype": "conference",
    "environment": "prod"
}
```

For more information about device twins and tags, see [Understand and use device twins in IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).

## Create a deployment

You deploy modules to your target devices by creating a deployment that consists of the deployment manifest as well as other parameters. 

Use the following command to create a deployment:

   ```cli
   az iot edge deployment create --deployment-id [deployment id] --hub-name [hub name] --content [file path] --labels "[labels]" --target-condition "[target query]" --priority [int]
   ```

* **--deployment-id** - The name of the deployment that will be created in the IoT hub. Give your deployment a unique name that is up to 128 lowercase letters. Avoid spaces and the following invalid characters: `& ^ [ ] { } \ | " < > /`.
* **--hub-name** - Name of the IoT hub in which the deployment will be created. The hub must be in the current subscription. Switch to the desired subscription with the command `az account set -s [subscription name]`.
* **--content** - Filepath to the deployment manifest JSON. 
* **--labels** - Add labels to help track your deployments. Labels are Name, Value pairs that describe your deployment. Labels take JSON formatting for the names and values. For example, `{"HostPlatform":"Linux", "Version:"3.0.1"}`
* **--target-condition** - Enter a target condition to determine which devices will be targeted with this deployment. The condition is based on device twin tags or device twin reported properties and should match the expression format. For example, `tags.environment='test' and properties.reported.devicemodel='4000x'`. 
* **--priority** - A positive integer. In the event that two or more deployments are targeted at the same device, the deployment with the highest numerical value for Priority will apply.

## Monitor a deployment

Use the following command to display the contents of a deployment:

   ```cli
az iot edge deployment show --deployment-id [deployment id] --hub-name [hub name]
   ```

* **--deployment-id** - The name of the deployment that exists in the IoT hub.
* **--hub-name** - Name of the IoT hub in which the deployment exists. The hub must be in the current subscription. Switch to the desired subscription with the command `az account set -s [subscription name]`

Inspect the deployment in the command window. The **metrics** property lists a count for each metric that is evaluated by each hub:

* **targetedCount** - A system metric that specifies the number of device twins in IoT Hub that match the targeting condition.
* **appliedCount** - A system metric specifies the number of devices that have had the deployment content applied to their module twins in IoT Hub.
* **reportedSuccessfulCount** - A device metric that specifies the number of Edge devices in the deployment reporting success from the IoT Edge client runtime.
* **reportedFailedCount** - A device metric that specifies the number of Edge devices in the deployment reporting failure from the IoT Edge client runtime.

You can show a list of device IDs or objects for each of the metrics by using the following command:

   ```cli
az iot edge deployment show-metric --deployment-id [deployment id] --metric-id [metric id] --hub-name [hub name] 
   ```

* **--deployment-id** - The name of the deployment that exists in the IoT hub.
* **--metric-id** - The name of the metric for which you want to see the list of device IDs, for example `reportedFailedCount`
* **--hub-name** - Name of the IoT hub in which the deployment exists. The hub must be in the current subscription. Switch to the desired subscription with the command `az account set -s [subscription name]`

## Modify a deployment

When you modify a deployment, the changes immediately replicate to all targeted devices. 

If you update the target condition, the following updates occur:

* If a device didn't meet the old target condition, but meets the new target condition and this deployment is the highest priority for that device, then this deployment is applied to the device. 
* If a device currently running this deployment no longer meets the target condition, it uninstalls this deployment and takes on the next highest priority deployment. 
* If a device currently running this deployment no longer meets the target condition and doesn't meet the target condition of any other deployments, then no change occurs on the device. The device continues running its current modules in their current state, but is not managed as part of this deployment anymore. Once it meets the target condition of any other deployment, it uninstalls this deployment and takes on the new one. 

Use the following command to update a deployment:

   ```cli
az iot edge deployment update --deployment-id [deployment id] --hub-name [hub name] --set [property1.property2='value']
   ```

* **--deployment-id** - The name of the deployment that exists in the IoT hub.
* **--hub-name** - Name of the IoT hub in which the deployment exists. The hub must be in the current subscription. Switch to the desired subscription with the command `az account set -s [subscription name]`
* **--set** - Update a property in the deployment. You can update the following properties:
  * targetCondition - for example `targetCondition=tags.location.state='Oregon'`
  * labels 
  * priority


## Delete a deployment

When you delete a deployment, any devices take on their next highest priority deployment. If your devices don't meet the target condition of any other deployment, then the modules are not removed when the deployment is deleted. 

Use the following command to delete a deployment:

   ```cli
az iot edge deployment delete --deployment-id [deployment id] --hub-name [hub name] 
   ```

* **--deployment-id** - The name of the deployment that exists in the IoT hub.
* **--hub-name** - Name of the IoT hub in which the deployment exists. The hub must be in the current subscription. Switch to the desired subscription with the command `az account set -s [subscription name]`

## Next steps

Learn more about [Deploying modules to Edge devices](module-deployment-monitoring.md).
