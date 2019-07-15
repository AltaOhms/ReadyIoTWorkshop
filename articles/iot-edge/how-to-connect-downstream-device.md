---
title: Connect downstream devices - Azure IoT Edge | Microsoft Docs
description: How to configure downstream or leaf devices to connect through Azure IoT Edge gateway devices. 
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 11/01/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom: seodec18
---

# Connect a downstream device to an Azure IoT Edge gateway

Azure IoT Edge enables transparent gateway scenarios, in which one or more devices can pass their messages through a single gateway device that maintains the connection to IoT Hub. Once you have the gateway device configured, you need to know how to securely connect the downstream devices. 

This article identifies common problems with downstream device connections and guides you in setting up your downstream devices by: 

* Explaining transport layer security (TLS) and certificate fundamentals. 
* Explaining how TLS libraries work across different operating systems and how each operating system deals with certificates.
* Walking through Azure IoT samples in several languages to help get you started. 

In this article, the terms *gateway* and *IoT Edge gateway* refer to an IoT Edge device configured as a transparent gateway. 

## Prerequisites

Before following the steps in this article, you should have two devices ready to use:

1. An IoT Edge device set up as a transparent gateway. 
    [Configure an IoT Edge device to act as a transparent gateway](how-to-create-transparent-gateway.md)

    Once you have your gateway device configured, copy the **azure-iot-test-only.root.ca.cert.pem** certificate from the gateway and have it available anywhere on your downstream device. 

2. A downstream device that has a device identity from IoT Hub. 
    You cannot use an IoT Edge device as the downstream device. Instead, use a device registered as a regular IoT device in IoT Hub. In the portal, you can register a new device in the **IoT devices** section. Or you can use the Azure CLI to [register a device](../iot-hub/quickstart-send-telemetry-c.md#register-a-device). Copy the connection string and have it available to use in later sections. 

    Currently, only downstream devices with symmetric key authentication can connect through IoT Edge gateways. X.509 certificate authorities and X.509 self-signed certificates are not currently supported.
    
> [!NOTE]
> The "gateway name" used in this article needs to be the same name as used as hostname in your IoT Edge config.yaml file. The gateway name needs to be resolvable to an IP Address, either using DNS or a host file entry. Communication based on the protocol used (MQTTS:8883/AMQPS:5671/HTTPS:433) must be possible between downstream device and the transparant IoT Edge. If a firewall is in between, the respective port needs to be open.

## Prepare a downstream device

A downstream device can be any application or platform that has an identity created with the [Azure IoT Hub](https://docs.microsoft.com/azure/iot-hub) cloud service. In many cases, these applications use the [Azure IoT device SDK](../iot-hub/iot-hub-devguide-sdks.md). For all practical purposes, a downstream device could even be an application running on the IoT Edge gateway device itself. 

To connect a downstream device to an IoT Edge gateway, you need two things:

1. A device or application that's configured with an IoT Hub device connection string appended with information to connect it to the gateway. 

    The connection string is formatted like: `HostName=yourHub.azure-devices.net;DeviceId=yourDevice;SharedAccessKey=XXXYYYZZZ=;`. Append the **GatewayHostName** property with the hostname of the gateway device to the end of the connection string. The value of **GatewayHostName** should match the value of **hostname** in the gateway device's config.yaml file. 

    The final string looks like: `HostName=yourHub.azure-devices.net;DeviceId=yourDevice;SharedAccessKey=XXXYYYZZZ=;GatewayHostName=mygateway.contoso.com`.

2. The device or application has to trust the gateway's **root CA** or **owner CA** certificate to validate the TLS connections to the gateway device. 

    This more complicated step is explained in detail in the rest of this article. This step can be performed one of two ways: by installing the CA certificate in the operating system's certificate store, or (for certain languages) referencing the certificate within applications using the Azure IoT SDKs.

## TLS and certificate fundamentals

The challenge of securely connecting downstream devices to IoT Edge is just like any other secure client/server communication that occurs over the internet. A client and a server securely communicate over the internet using [Transport layer security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS is built using standard [Public key infrastructure (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure) constructs called certificates. TLS is a fairly involved specification and addresses a wide range of topics related to securing two endpoints but the following section concisely describes what is needed to securely connect devices to an IoT Edge gateway.

When a client connects to a server, the server presents a chain of certificates, called the *server certificate chain*. A certificate chain typically comprises a root certificate authority (CA) certificate, one or more intermediate CA certificates, and finally the server's certificate itself. A client establishes trust with a server by cryptographically verifying the entire server certificate chain. This client validation of the server certificate chain is called *server authentication*. To validate a server certificate chain, a client needs a copy of the root CA certificate that was used to create (or issue) the server's certificate. Normally when connecting to websites, a browser comes pre-configured with commonly used CA certificates so the client has a seamless process. 

When a device connects to Azure IoT Hub, the device is the client and the IoT Hub cloud service is the server. The IoT Hub cloud service is backed by a root CA certificate called **Baltimore CyberTrust Root**, which is publicly available and widely used. Since the IoT Hub CA certificate is already installed on most devices, many TLS implementations (OpenSSL, Schannel, LibreSSL) automatically use it during server certificate validation. A device that may successfully connect to IoT Hub may have issues trying to connect to an IoT Edge gateway.

When a device connects to an IoT Edge gateway, the downstream device is the client and the gateway device is the server. Azure IoT Edge allows operators (or users) to build gateway certificate chains however they see fit. The operator may choose to use a public CA certificate, like Baltimore, or use a self-signed (or in-house) root CA certificate. Public CA certificates often have a cost associated with them, so are typically used in production scenarios. Self-signed CA certificates are preferred for development and testing. The transparent gateway setup articles listed in the prerequisites section use self-signed root CA certificates. 

When you use a self-signed root CA certificate for an IoT Edge gateway, it needs to be installed on or provided to all the downstream devices attempting to connect to the gateway. 

To learn more about IoT Edge certificates and some production implications, see [IoT Edge certificate usage details](iot-edge-certs.md).

## Install certificates using the OS

This article uses *owner CA* to refer to the root CA certificate, since that's the term used by the scripts in the prerequisite gateway article. 

Installing the owner CA certificate in the operating system's certificate store generally allows most applications to use the owner CA certificate. There are some exceptions, like NodeJS application which don't use the OS certificate store but rather use the Node runtime's internal certificate store. If you can't install the certificate at the operating system level, refer to the language-specific examples later in this article to use the certificate with the Azure IoT SDK in applications. 

### Ubuntu

The following commands are an example of how to install a CA certificate on an Ubuntu host. This sample assumes that you're using the **azure-iot-test-only.root.ca.cert.pem** certificate from the prerequisites articles, and that you've copied the certificate into a location on the downstream device.  

```bash
sudo cp <path>/azure-iot-test-only.root.ca.cert.pem /usr/local/share/ca-certificates/azure-iot-test-only.root.ca.cert.pem.crt
sudo update-ca-certificates
```

You should see a message that says, "Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done."

### Windows

The following steps are an example of how to install a CA certificate on a Windows host. This sample assumes that you're using the **azure-iot-test-only.root.ca.cert.pem** certificate from the prerequisites articles, and that you've copied the certificate into a location on the downstream device.  

1. In the Start menu, search for and select **Manage computer certificates**. A utility called **certlm** opens.
2. Navigate to **Certificates - Local Computer** > **Trusted Root Certification Authorities**.
3. Right-click **Certificates** and select **All Tasks** > **Import**. The certificate import wizard should launch. 
4. Follow the steps as directed and import certificate file `<path>/azure-iot-test-only.root.ca.cert.pem`. When completed, you should see a "Successfully imported" message. 

You can also install certificates programmatically using .NET APIs, as shown in the .NET sample later in this article. 

Typically applications use the Windows provided TLS stack called [Schannel](https://docs.microsoft.com/windows/desktop/com/schannel) to securely connect over TLS. Schannel *requires* that any certificates be installed in the Windows certificate store before attempting to establish a TLS connection.

## Use certificates with Azure IoT SDKs

This article refers to the root CA certificate as the *owner CA* since that's the term used by the scripts that generate the self-signed certificate in the prerequisites articles. 

This section describes how the Azure IoT SDKs connect to an IoT Edge device using simple sample applications. The goal of all the samples is to connect the device client and send telemetry messages to the gateway, then close the connection and exit. 

### Common concepts across all Azure IoT SDKs

Have two things ready before using the application-level samples:

1. Your downstream device's IoT Hub connection string modified to point to the gateway device.

    The connection string is formatted like: `HostName=yourHub.azure-devices.net;DeviceId=yourDevice;SharedAccessKey=XXXYYYZZZ=;`. Append the **GatewayHostName** property with the hostname of the gateway device to the end of the connection string. The value of **GatewayHostName** should match the value of **hostname** in the gateway device's config.yaml file. 

    The final string looks like: `HostName=yourHub.azure-devices.net;DeviceId=yourDevice;SharedAccessKey=XXXYYYZZZ=;GatewayHostName=mygateway.contoso.com`.

2. The full path to the root CA certificate that you copied and saved somewhere on your downstream device.

    For example, `<path>/azure-iot-test-only.root.ca.cert.pem`. 

### NodeJS

This section provides a sample application to connect an Azure IoT NodeJS device client to an IoT Edge gateway. For Linux and Windows hosts, you must install the root CA certificate at the application level as shown here, as NodeJS applications don't use the system's certificate store. 

1. Get the sample for **edge_downstream_device.js** from the [Azure IoT device SDK for Node.js samples repo](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples). 
2. Make sure that you have all the prerequisites to run the sample by reviewing the **readme.md** file. 
3. In the edge_downstream_device.js file, update the **connectionString** and **edge_ca_cert_path** variables. 
4. Refer to the SDK documentation for instructions on how to run the sample on your device. 

To understand the sample that you're running, the following code snippet is how the client SDK reads the certificate file and uses it to establish a secure TLS connection: 

```javascript
// Provide the Azure IoT device client via setOptions with the X509
// Edge root CA certificate that was used to setup the Edge runtime
var options = {
    ca : fs.readFileSync(edge_ca_cert_path, 'utf-8'),
};
```

### .NET

This section introduces a sample application to connect an Azure IoT .NET device client to an IoT Edge gateway. However, .NET applications are automatically able to use any installed certificates in the system's certificate store on both Linux and Windows hosts.

1. Get the sample for **EdgeDownstreamDevice** from the [IoT Edge .NET samples folder](https://github.com/Azure/iotedge/tree/master/samples/dotnet/EdgeDownstreamDevice). 
2. Make sure that you have all the prerequisites to run the sample by reviewing the **readme.md** file. 
3. In the **Properties / launchSettings.json** file, update the **DEVICE_CONNECTION_STRING** and **CA_CERTIFICATE_PATH** variables. If you want to use the certificate installed in the trusted certificate store on the host system, leave this variable blank. 
4. Refer to the SDK documentation for instructions on how to run the sample on your device. 

To programmatically install a trusted certificate in the certificate store via a .NET application, refer to the **InstallCACert()** function in the **EdgeDownstreamDevice / Program.cs** file. This operation is idempotent, so can be run multiple times with the same values with no additional effect. 

### C

This section introduces a sample application to connect an Azure IoT C device client to an IoT Edge gateway. The C SDK can operate with many TLS libraries, including OpenSSL, WolfSSL, and Schannel. For more information, see the [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c). 

1. Get the **iotedge_downstream_device_sample** application from the [Azure IoT device SDK for C samples](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples). 
2. Make sure that you have all the prerequisites to run the sample by reviewing the **readme.md** file. 
3. In the iotedge_downstream_device_sample.c file, update the **connectionString** and **edge_ca_cert_path** variables. 
4. Refer to the SDK documentation for instructions on how to run the sample on your device. 

The Azure IoT device SDK for C provides an option to register a CA certificate when setting up the client. This operation doesn't install the certificate anywhere, but rather uses a string format of the certificate in memory. The saved certificate is provided to the underlying TLS stack when establishing a connection. 

```C
(void)IoTHubDeviceClient_SetOption(device_handle, OPTION_TRUSTED_CERT, cert_string);
```

On Windows hosts, if you're not using OpenSSL or another TLS library, the SDK default to using Schannel. For Schannel to work, the IoT Edge root CA certificate should be installed in the Windows certificate store, not set using the `IoTHubDeviceClient_SetOption` operation. 

### Java

This section introduces a sample application to connect an Azure IoT Java device client to an IoT Edge gateway. 

1. Get the sample for **Send-event** from the [Azure IoT device SDK for Java samples](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples). 
2. Make sure that you have all the prerequisites to run the sample by reviewing the **readme.md** file. 
3. Refer to the SDK documentation for instructions on how to run the sample on your device.

### Python

This section introduces a sample application to connect an Azure IoT Python device client to an IoT Edge gateway. 

1. Get the sample for **edge_downstream_client** from the [Azure IoT device SDK for Python samples](https://github.com/Azure/azure-iot-sdk-python/tree/master/device/samples). 
2. Make sure that you have all the prerequisites to run the sample by reviewing the **readme.md** file. 
3. In the edge_downstream_client.py file, update the **CONNECTION_STRING** and **TRUSTED_ROOT_CA_CERTIFICATE_PATH** variables. 
4. Refer to the SDK documentation for instructions on how to run the sample on your device. 


## Test the gateway connection

This is a sample command which tests that everything has been set up correctly. You should see a message saying "verified OK".

```cmd/sh
openssl s_client -connect mygateway.contoso.com:8883 -CAfile <CERTDIR>/certs/azure-iot-test-only.root.ca.cert.pem -showcerts
```

## Troubleshoot the gateway connection

If your leaf device has intermittent connection to its gateway device, try the following steps for resolution. 

1. Is the gateway name appended to the connection string the same as the hostname in the IoT Edge config.yaml file on the gateway device?
2. Is the gateway name resolvable to an IP Address? You can resolve intenmittent connections either by using DNS or by adding a host file entry on the leaf device.
3. Are communication ports open in your firewall? Communication based on the protocol used (MQTTS:8883/AMQPS:5671/HTTPS:433) must be possible between downstream device and the transparant IoT Edge.

## Next steps

Learn how IoT Edge can extend [offline capabilities](offline-capabilities.md) to downstream devices. 
