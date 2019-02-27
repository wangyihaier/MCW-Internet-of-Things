# Internet of Things

Fabrikam provides services and smart meters for enterprise energy (electrical power) management. Their “You-Left-The-Light-On” service enables the enterprise to understand their energy consumption. Fabrikam would like to become an authorized energy management solution provider. According to their Director of Analytics, Sam George, "We are investigating a move to the cloud to help our customers not only to meet data collection and reporting requirements, but also become the number one energy management solution provider." They are intending to enable their enterprise customers with a web-based dashboard where they can see historical trends of power consumption.

## Target audience

- Application developer
- IoT

## Abstract

### Workshop

This workshop will guide you through an implementation of an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path.

At the end of this workshop, you will be better able to construct an IoT solution implementing device registration with the IoT Hub Device Provisioning Service and visualizing hot data with Power BI.

### Hands-on labs

#### Lab1 : Pirmary IoT Solution with hot path and cold path analytics

**Duration**: 5 hours

[Internet of Things hands-on lab step-by-step](Hands-on%20lab/Before%20the%20HOL%20-%20Internet%20of%20Things.md)

In this hands-on lab, you will construct an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path.

#### Lab2 : Create Azure Time Series Insights and Visualize Device Data

**Duration**: 60 minutes

[Time Series Insights Lab](Hands-on%20lab/HOL%20step-by-step-timeseriesinsights.md)

#### Lab3 : Capture Device Events and Send Notifications

**Duration**: 40 minutes

[Azure IoTHub with Event Grid Lab](Hands-on%20lab/HOL%20step-by-step-EventGrids.md)

#### Lab4 : Save the Sensor Data using Serverless Archiecture and CosmosDB

**Duration**: 100 minutes

[Azure Serverless Architecture](Hands-on%20lab/HOL%20step-by-step%20-%20Serverless%20architecture.md)


## Azure services and related products

- Azure App Services
- Azure Blob Storage
- Azure Data Factory
- Azure Databricks
- Azure SQL Database
- Azure Stream Analytics
- IoT Hub
- Azure Functions
- Azure Logic App
- Time Series Insights
- Power BI Desktop
- Visual Studio 2017

## Azure solution

Internet of Things

## Related references

[MCW](https://github.com/Microsoft/MCW)
