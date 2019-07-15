---
title: Real-time data visualization of sensor data from Azure IoT Hub – Power BI | Microsoft Docs
description: Use Power BI to visualize temperature and humidity data that is collected from the sensor and sent to your Azure IoT hub.
author: robinsh
keywords: real time data visualization, live data visualization, sensor data visualization
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 4/11/2018
ms.author: robinsh
---

# Visualize real-time sensor data from Azure IoT Hub using Power BI

![End-to-end diagram](./media/iot-hub-live-data-visualization-in-power-bi/1_end-to-end-diagram.png)


[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

## What you learn

You learn how to visualize real-time sensor data that your Azure IoT hub receives by using Power BI. If you want to try toe visualize the data in your IoT hub with Web Apps, please see [Use Azure Web Apps to visualize real-time sensor data from Azure IoT Hub](iot-hub-live-data-visualization-in-web-apps.md).

## What you do

* Get your IoT hub ready for data access by adding a consumer group.

* Create, configure, and run a Stream Analytics job for data transfer from your IoT hub to your Power BI account.

* Create and publish a Power BI report to visualize the data.

## What you need

* Complete the [Raspberry Pi online simulator](iot-hub-raspberry-pi-web-simulator-get-started.md) tutorial or one of the device tutorials; for example, [Raspberry Pi with node.js](iot-hub-raspberry-pi-kit-node-get-started.md). These cover the following requirements:
  
  * An active Azure subscription.
  * An Azure IoT hub under your subscription.
  * A client application that sends messages to your Azure IoT hub.

* A Power BI account. ([Try Power BI for free](https://powerbi.microsoft.com/))

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## Create, configure, and run a Stream Analytics job

Let's start by creating a Stream Analytics job. After you create the job, you define the inputs, outputs, and the query used to retrieve the data.

### Create a Stream Analytics job

1. In the [Azure portal](https://portal.azure.com), click **Create a resource** > **Internet of Things** > **Stream Analytics job**.

2. Enter the following information for the job.

   **Job name**: The name of the job. The name must be globally unique.

   **Resource group**: Use the same resource group that your IoT hub uses.

   **Location**: Use the same location as your resource group.

   **Pin to dashboard**: Check this option for easy access to your IoT hub from the dashboard.

   ![Create a Stream Analytics job in Azure](./media/iot-hub-live-data-visualization-in-power-bi/2_create-stream-analytics-job-azure.png)

3. Click **Create**.

### Add an input to the Stream Analytics job

1. Open the Stream Analytics job.

2. Under **Job Topology**, click **Inputs**.

3. In the **Inputs** pane, click **Add stream input**, and then enter the following information:

   **Input alias**: The unique alias for the input and select **Provide IoT Hub settings manually** below.

   **Source**: Select **IoT hub**.
   
   **Endpoint**: Click **Messaging**.

   **Consumer group**: Select the consumer group you just created.

4. Click **Create**.

   ![Add an input to a Stream Analytics job in Azure](./media/iot-hub-live-data-visualization-in-power-bi/3_add-input-to-stream-analytics-job-azure.png)

### Add an output to the Stream Analytics job

1. Under **Job Topology**, click **Outputs**.

2. In the **Outputs** pane, click **Add** and **Power BI**, and then enter the following information:

   **Output alias**: The unique alias for the output.

   **Group Workspace**: Select your target group workspace.

   **Dataset Name**: Enter a dataset name.

   **Table Name**: Enter a table name.

3. Click **Authorize**, and then sign into your Power BI account.

4. Click **Create**.

   ![Add an output to a Stream Analytics job in Azure](./media/iot-hub-live-data-visualization-in-power-bi/4_add-output-to-stream-analytics-job-azure.png)

### Configure the query of the Stream Analytics job

1. Under **Job Topology**, click **Query**.

2. Replace `[YourInputAlias]` with the input alias of the job.

3. Replace `[YourOutputAlias]` with the output alias of the job.

4. Click **Save**.

   ![Add a query to a Stream Analytics job in Azure](./media/iot-hub-live-data-visualization-in-power-bi/5_add-query-stream-analytics-job-azure.png)

### Run the Stream Analytics job

In the Stream Analytics job, click **Start** > **Now** > **Start**. Once the job successfully starts, the job status changes from **Stopped** to **Running**.

![Run a Stream Analytics job in Azure](./media/iot-hub-live-data-visualization-in-power-bi/6_run-stream-analytics-job-azure.png)

## Create and publish a Power BI report to visualize the data

1. Ensure the sample application is running on your device. If not, you can refer to the tutorials under [Setup your device](https://docs.microsoft.com/azure/iot-hub/iot-hub-raspberry-pi-kit-node-get-started).

2. Sign in to your [Power BI](https://powerbi.microsoft.com/en-us/) account.

3. Click the workspace you used, **My Workspace**.

4. Click **Datasets**.

   You should see the dataset that you specified when you created the output for the Stream Analytics job.

5. For the dataset you created, click **Add Report** (the first icon to the right of the dataset name).

   ![Create a Microsoft Power BI report](./media/iot-hub-live-data-visualization-in-power-bi/7_create-power-bi-report-microsoft.png)

6. Create a line chart to show real-time temperature over time.

   1. On the report creation page, add a line chart.

   2. On the **Fields** pane, expand the table that you specified when you created the output for the Stream Analytics job.
   
   3. Drag **EventEnqueuedUtcTime** to **Axis** on the **Visualizations** pane.
   
   4. Drag **temperature** to **Values**.

      A line chart is created. The x-axis displays date and time in the UTC time zone. The y-axis displays temperature from the sensor.

      ![Add a line chart for temperature to a Microsoft Power BI report](./media/iot-hub-live-data-visualization-in-power-bi/8_add-line-chart-for-temperature-to-power-bi-report-microsoft.png)

7. Create another line chart to show real-time humidity over time. To do this, follow the same steps above and place **EventEnqueuedUtcTime** on the x-axis and **humidity** on the y-axis.

   ![Add a line chart for humidity to a Microsoft Power BI report](./media/iot-hub-live-data-visualization-in-power-bi/9_add-line-chart-for-humidity-to-power-bi-report-microsoft.png)

8. Click **Save** to save the report.

9. Click **Reports** on the left pane, and then click the report that you just created.

10. Click **File** > **Publish to web**.

11. Click **Create embed code**, and then click **Publish**.

You're provided the report link that you can share with anyone for report access and a code snippet to integrate the report into your blog or website.

![Publish a Microsoft Power BI report](./media/iot-hub-live-data-visualization-in-power-bi/10_publish-power-bi-report-microsoft.png)

Microsoft also offers the [Power BI mobile apps](https://powerbi.microsoft.com/en-us/documentation/powerbi-power-bi-apps-for-mobile-devices/) for viewing and interacting with your Power BI dashboards and reports on your mobile device.

## Next steps

You’ve successfully used Power BI to visualize real-time sensor data from your Azure IoT hub.

There is an alternate way to visualize data from Azure IoT Hub. See [Use Azure Web Apps to visualize real-time sensor data from Azure IoT Hub](iot-hub-live-data-visualization-in-web-apps.md).

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
