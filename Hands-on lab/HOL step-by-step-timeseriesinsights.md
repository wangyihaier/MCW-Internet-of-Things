# Create Azure Time Series Insights and Visualize Device Data

![Time Series Insights](timeseriesinsights/images/timeseriesinsights.jpg)

## Create Time Series Insights

Azure Time Series Insights is a fully managed analytics, storage, and visualization service for managing IoT-scale time-series data in the cloud. It provides massively scalable time-series data storage and enables you to explore and analyze billions of events streaming in from all over the world in seconds. Use Time Series Insights to store and manage terabytes of time-series data, explore and visualize billions of events simultaneously, conduct root-cause analysis, and to compare multiple sites and assets.

Time Series Insights has four key jobs:

* First, it's fully integrated with cloud gateways like Azure IoT Hub and Azure Event Hubs. It easily connects to these event sources and parses JSON from messages and structures that have data in clean rows and columns. It joins metadata with telemetry and indexes your data in a columnar store.
* Second, Time Series Insights manages the storage of your data. To ensure data is always easily accessible, it stores your data in memory and SSD’s for up to 400 days. You can interactively query billions of events in seconds – on demand.
* Third, Time Series Insights provides out-of-the-box visualization via the TSI explorer. 
* Fourth, Time Series Insights provides a query service, both in the TSI explorer and by using APIs that are easy to integrate for embedding your time series data into custom applications.

<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Time-Series-Insight-for-IoT-apps/player" width="480" height="270" allowFullScreen frameBorder="0"></iframe>

In this lab you will learn

* how to set up a Time Series Insights environment
* explore
* analyze time series data of your IoT solutions or connected things


Click on **Create a Resource** and click on **Internet of Things**

![Create Time Series Insights](timeseriesinsights/images/01_Create_Time_Series_Insights.png)

Click on **Time Series Insights**

![Create Time Series Insights](timeseriesinsights/images/tsi.png)

Select the resource group you previously created and click **Next:Event Source>>** button

![Create Time Series Insights Submit](timeseriesinsights/images/02_Create_Time_Series_Inisghts_Submit_New.png)

### Create Event Source

Create Event Source to connect to IoTHub. Please make sure you use a unique Consumer Group. Time Series Insights has a requirement to have its own unique consumer group

Provide the inputs for the mandatory fields and click **Review + create** button

![Create Event Source Submit](timeseriesinsights/images/04_Create_Event_Source_Submit_New.png)

### Setup Time Series Insights

Go To Time Series Insights, Click on Go To Environment which will take you to Time Series Insights Explorer

If you get Data Access Policy Error execute the following steps

![Data Access Policy Error](timeseriesinsights/images/16_data_access_poliy_error.png)

Go To Environment Topology and 

![Select Data Access Policy](timeseriesinsights/images/15_data_access_policy.png)

Click on Add Button

![Add User and Role](timeseriesinsights/images/17_add_user_role.png)

Select Contributor Role

![Select Contributor Role](timeseriesinsights/images/18_select_controbutor_role.png)

Select User

![Select User](timeseriesinsights/images/19_select_user.png)

### Time Series Insights Explorer

Go To Time Series Insights Explorer

![Visualize Data](timeseriesinsights/images/05_GoTo_TSI_Explorer.png)

Split By ID. You will see data flowing from three devices. 

![Visualize Data](timeseriesinsights/images/06_Visual1_New.png)

Select humidity and Split By ID. You will see data flowing from three devices. 

![Visualize Data](timeseriesinsights/images/07_Visual2_New.png)

Right Click to Explore events. You can download events in CSV and JSON format by clicking on **CSV or JSON** buttons

![Visualize Data](timeseriesinsights/images/08_Visual3_New.png)

Create a perspective by clicking on the image shown below

![Visualize Data](timeseriesinsights/images/perspective.png)

Click **+** to add a new query

![Visualize Data](timeseriesinsights/images/10_visual10_New.png)

Select Events and split by Device ID and click on perspective image.

![Visualize Data](timeseriesinsights/images/11_visual11_New.png)

Create a chart by selecting a timeframe with drag feature

![Visualize Data](timeseriesinsights/images/12_Visual12.png)

Create a Chart by adding a predicate

![Visualize Data](timeseriesinsights/images/predicate_New.png)

Perspective with 4 different charts and also changed Title

![Visualize Data](timeseriesinsights/images/14_Visual_dashboard_New.png)

Click on Heatmap

![Visualize Data](timeseriesinsights/images/heatmap_New.png)

View data in a table

![Visualize Data](timeseriesinsights/images/table_New.png)
