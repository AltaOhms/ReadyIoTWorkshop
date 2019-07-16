# Visualize Time Series Data with Azure Time Series Insights

![Time Series Insights](images/TSI-Lab/timeseriesinsights.jpg)

In this lab you will learn how to

* set up a Time Series Insights environment
* explore and analyze time series data of your IoT solutions or connected things
Click on **Create a Resource** and click on **Internet of Things**

![Create Time Series Insights](images/TSI-Lab/01_Create_Time_Series_Insights.png)

Click on **Time Series Insights**

![Create Time Series Insights](images/TSI-Lab/tsi.png)

Select the resource group you previously created, and fill in the other parameters. Then click **Next: Event Source**.

![Create Time Series Insights Basics](images/TSI-Lab/create-tsi-basics.png)

Click **New** to add an IoT Hub Consumer Group with a descriptive name, for example **"timeseriesinsightsevents"**. Then click **Review + create**. After reviewing, click **Create**.

![Create Time Series Insights Event Source](images/TSI-Lab/create-tsi-eventsource.png)

### Setup Time Series Insights

Go To Time Series Insights, Click on Go To Environment which will take you to Time Series Insights Explorer

![Go To Environment](images/TSI-Lab/go-to-environment.png)

If you get a Data Access Policy Error execute the following steps.

Go To Data Access Policies

![Select Data Access Policy](images/TSI-Lab/15_data_access_policy.png)

Click on Add Button

![Add User and Role](images/TSI-Lab/17_add_user_role.png)

Select Contributor Role

![Select Contributor Role](images/TSI-Lab/18_select_controbutor_role.png)

Select User

![Select User](images/TSI-Lab/19_select_user.png)

### Time Series Insights Explorer

Go To Time Series Insights Explorer

Select humidity. You will see data flowing from your MXChip device. 

![Visualize Data](images/TSI-Lab/tsi-humidity.png)

Right Click to Explore events. You can download events in CSV and JSON format by clicking on **CSV or JSON** buttons

![Visualize Data](images/TSI-Lab/tsi-explore-events.png)

Create a perspective by clicking on the image shown below

![Visualize Data](images/TSI-Lab/perspective.png)

Click **+** to add a new query

![Visualize Data](images/TSI-Lab/10_visual10.png)

Create a chart by selecting a timeframe with drag feature

![Visualize Data](images/TSI-Lab/12_Visual12.png)

This time set the **Measure** to **Temperature**. Click on perspective image.

![Visualize Data](images/TSI-Lab/add-query.png)

Click on Heatmap and save as another chart.

![Visualize Data](images/TSI-Lab/heatmap.png)

Perspective with 4 different charts and custom Title

![Visualize Data](images/TSI-Lab/4-charts.png)

View data in a table. If you have multiple devices, they will appear as separate rows in the table as below.

![Visualize Data](images/TSI-Lab/table.png)

## Finished!

You have successfully set up Azure Time Series Insights and analyzed your device's time series data.

## Optional Chellenger Lab 

### How to Incorporate Time Series Insight Data into Your Own App? 

>[!Tip]
> - TSI data can be accessed via REST API
> - Leverage Azure AD to grant permissions to TSI API

Reference: 
- <https://docs.microsoft.com/en-us/azure/time-series-insights/tutorial-create-tsi-sample-spa>
- <https://docs.microsoft.com/en-us/rest/api/time-series-insights/ga-query-api>
