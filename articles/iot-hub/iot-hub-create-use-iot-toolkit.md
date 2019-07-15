# Create an IoT hub using the Azure IoT Tools for Visual Studio Code

This article shows you how to use the [Azure IoT Tools for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) to create an Azure IoT hub. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

To complete this article, you need the following:

- An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

- [Visual Studio Code](https://code.visualstudio.com/)

- [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) for Visual Studio Code.

## Create an IoT hub

1. In Visual Studio Code, open the **Explorer** view.

2. At the bottom of the Explorer, expand the **Azure IoT Hub Devices** section. 

   ![Expand Azure IoT Hub Devices](./media/iot-hub-create-use-iot-toolkit/azure-iot-hub-devices.png)

3. Click on the **...** in the **Azure IoT Hub Devices** section header. If you don't see the ellipsis, hover over the header.

4. Choose **Create IoT Hub**.

5. A pop-up will show in the bottom right corner to let you sign in to Azure for the first time.

6. Select Azure subscription. 

7. Select resource group.

8. Select location.

9. Select pricing tier.

10. Enter a globally unique name for your IoT Hub.

11. Wait a few minutes until the IoT Hub is created.
