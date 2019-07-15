# Install the Azure IoT Edge runtime on Linux (x64)

The Azure IoT Edge runtime is what turns a device into an IoT Edge device. The runtime can be deployed on devices as small as a Raspberry Pi or as large as an industrial server. Once a device is configured with the IoT Edge runtime, you can start deploying business logic to it from the cloud.

To learn more, see [Understand the Azure IoT Edge runtime and its architecture](iot-edge-runtime.md).

This article lists the steps to install the Azure IoT Edge runtime on your Ubuntu Linux x64 (Intel/AMD) IoT Edge device. Refer to [Azure IoT Edge support](support.md#operating-systems) for a list of supported AMD64 operating systems.

> [!NOTE]
> Packages in the Linux software repositories are subject to the license terms located in each package (/usr/share/doc/*package-name*). Read the license terms prior to using the package. Your installation and use of the package constitutes your acceptance of these terms. If you do not agree with the license terms, do not use the package.

## Register Microsoft key and software repository feed

Prepare your device for the IoT Edge runtime installation.

Install the repository configuration for **Ubuntu 18.04**:

```bash
sudo apt-get update && \
sudo apt-get install -y curl && \
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > ./microsoft-prod.list && \
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/ && \
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg && \
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/

```

## Install the container runtime

Azure IoT Edge relies on an [OCI-compatible](https://www.opencontainers.org/) container runtime. For production scenarios, we recommended that you use the [Moby-based](https://mobyproject.org/) engine provided below. It is the only container engine officially supported with Azure IoT Edge. Docker CE/EE container images are compatible with the Moby runtime.

```bash
sudo apt-get update && \
sudo apt-get install -y moby-engine && \
sudo apt-get install -y moby-cli

```

## Install the Azure IoT Edge Security Daemon

The **IoT Edge security daemon** provides and maintains security standards on the IoT Edge device. The daemon starts on every boot and bootstraps the device by starting the rest of the IoT Edge runtime.

The installation command also installs the standard version of the **iothsmlib** if not already present.

```bash
sudo apt-get update && \
sudo apt-get install -y iotedge

```

## Configure the Azure IoT Edge Security Daemon

Configure the IoT Edge runtime to link your physical device with a device identity that exists in an Azure IoT hub.

The daemon can be configured using the configuration file at `/etc/iotedge/config.yaml`. The file is write-protected by default, you might need elevated permissions to edit it.

For this lab, we will manually provision a new device.

### Manual provisioning

To manually provision a device, you need to provide it with a [device connection string](how-to-register-device-portal.md) that you can create by registering a new device in your IoT hub.

Open the configuration file.

```bash
sudo nano /etc/iotedge/config.yaml
```

Find the provisioning section of the file. Uncomment the **manual** provisioning mode, and make sure the dps provisioning mode is commented out. Update the value of **device_connection_string** with the connection string from your IoT Edge device.

```yaml
provisioning:
   source: "manual"
   device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"

# provisioning:
#   source: "dps"
#   global_endpoint: "https://global.azure-devices-provisioning.net"
#   scope_id: "{scope_id}"
#   registration_id: "{registration_id}"
```

Save and close the file.

`CTRL + X`, `Y`, `Enter`

After entering the provisioning information in the configuration file, restart the daemon:

```bash
sudo systemctl restart iotedge
```

## Verify successful installation

You can check the status of the IoT Edge Daemon using:

```bash
systemctl status iotedge
```

Examine daemon logs using:

```bash
journalctl -u iotedge --no-pager --no-full
```

And, list running modules with:

```bash
sudo iotedge list
```

## Tips and suggestions

You need elevated privileges to run `iotedge` commands. After installing the runtime, sign out of your machine and sign back in to update your permissions automatically. Until then, use **sudo** in front of any `iotedge` the commands.

### Uninstall IoT Edge

If you want to remove the IoT Edge installation from your Linux device, use the following commands from the command line.

Remove the IoT Edge runtime.

```bash
sudo apt-get remove --purge iotedge
```

You can clean up the containers.  This will delete all containers and network resources created.

```bash
sudo docker system prune -a
```

Finally, remove the container runtime from your device.

```bash
sudo apt-get remove --purge moby-cli
sudo apt-get remove --purge moby-engine
```
