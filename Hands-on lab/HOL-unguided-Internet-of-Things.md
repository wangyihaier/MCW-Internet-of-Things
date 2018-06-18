![Microsoft Cloud Workshop](../media/ms-cloud-workshop.png "Microsoft Cloud Workshop")

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.
© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

# Internet of Things hands-on lab unguided

Updated June 2018

In this hands-on lab, you will implement an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path. After completing the hands-on lab, you will have a better understanding of implementing device registration with the IoT Hub Device Provisioning Service and visualizing hot data with Power BI.

If you have not yet completed the steps to set up your environment in [Before the hands-on lab](./Setup.md), you will need to do that before proceeding.

## Contents

* [Abstract](#abstract)
* [Overview](#overview)
* [Solution architecture](#solution-architecture)
* [Requirements](#requirements)
* [Exercise 1: Environment setup](#exercise-1-environment-setup)
  * [Task 1: Download and open the Smart Meter Simulator project](#task-1-download-and-open-the-smart-meter-simulator-project)
* [Exercise 2: IoT Hub provisioning](#exercise-2-iot-hub-provisioning)
  * [Task 1: Provision an IoT Hub](#task-1-provision-an-iot-hub)
  * [Task 2: Configure the Smart Meter Simulator](#task-2-configure-the-smart-meter-simulator)
* [Exercise 3: Completing the Smart Meter Simulator](#exercise-3-completing-the-smart-meter-simulator)
  * [Task 1: Implement device management with the IoT Hub](#task-1-implement-device-management-with-the-iot-hub)
  * [Task 2: Implement the communication of telemetry with the IoT Hub](#task-2-implement-the-communication-of-telemetry-with-the-iot-hub)
  * [Task 3: Verify device registration and telemetry](#task-3-verify-device-registration-and-telemetry)
* [Exercise 4: Hot path data processing with Stream Analytics](#exercise-4-hot-path-data-processing-with-stream-analytics)
  * [Task 1: Create a Stream Analytics job for hot path processing to Power BI](#task-1-create-a-stream-analytics-job-for-hot-path-processing-to-power-bi)
  * [Task 2: Visualize hot data with Power BI](#task-2-visualize-hot-data-with-power-bi)
* [Exercise 5: Cold path data processing with HDInsight Spark](#exercise-5-cold-path-data-processing-with-hdinsight-spark)
  * [Task 1: Create the Stream Analytics job for cold path processing](#task-1-create-the-stream-analytics-job-for-cold-path-processing)
  * [Task 2: Verify CSV files in blob storage](#task-2-verify-csv-files-in-blob-storage)
  * [Task 3: Update pandas version on the Spark cluster](#task-3-update-pandas-version-on-the-spark-cluster)
  * [Task 4: Process with Spark SQL](#task-4-process-with-spark-sql)
* [Exercise 6: Reporting device outages with IoT Hub Operations Monitoring](#exercise-6-reporting-device-outages-with-iot-hub-operations-monitoring)
  * [Task 1: Enable verbose connection monitoring on the IoT Hub](#task-1-enable-verbose-connection-monitoring-on-the-iot-hub)
  * [Task 2: Collect device connection telemetry with the hot path Stream Analytics job](#task-2-collect-device-connection-telemetry-with-the-hot-path-stream-analytics-job)
  * [Task 3: Test the device outage notifications](#task-3-test-the-device-outage-notifications)
  * [Task 4: Visualize disconnected devices with Power BI](#task-4-visualize-disconnected-devices-with-power-bi)
* [After the hands-on lab](#after-the-hands-on-lab)
  * [Task 1: Delete the resource group](#task-1-delete-the-resource-group)

## Abstract

In this hands-on lab, you will construct an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path.

At the end of this hands-on lab, you will be better able to build an IoT solution implementing device registration with the IoT Hub Device Provisioning Service and visualizing hot data with Power BI.

## Overview

Fabrikam provides services and smart meters for enterprise energy (electrical power) management. Their "*You-Left-The-Light-On*" service enables the enterprise to understand their energy consumption.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

![Diagram of the preferred solution. From a high-level, the commerce solution uses an API App to host the Payments web service with which the Vending Machine interacts to conduct purchase transactions. The Payment Web API invokes a 3rd party payment gateway as needed for authorizing and capturing credit card payments, and logs the purchase transaction to SQL DB. The data for these purchase transactions is stored using an In-Memory table with a Columnar Index, which will support the write-heavy workload while still allowing analytics to operate, such as queries coming from Power BI Desktop.](./media/preferred-solution-architecture.png "Preferred high-level architecture")

## Requirements

* Microsoft Azure subscription must be pay-as-you-go or MSDN
  * Trial subscriptions will not work
* A virtual machine configured with:
  * Visual Studio Community 2017 15.6 or later
  * Azure SDK 2.9 or later (Included with Visual Studio 2017)
* A running Azure Databricks cluster (see [Before the hands-on lab](#before-the-hands-on-lab))

## Exercise 2: IoT Hub provisioning

Duration: 20 minutes

In your architecture design session with Fabrikam, it was agreed that you would use an Azure IoT Hub to manage both the device registration and telemetry ingest from the Smart Meter Simulator. Your team also identified the Microsoft provided Device Explorer project that Fabrikam can use to view the list and status of devices in the IoT Hub registry.

### Task 1: Provision an IoT Hub

In these steps, you will provision an instance of IoT Hub.

#### Tasks to complete

* Provision an IoT Hub instance.

* Determine and take note of the connection strings required for 1) full control and 2) read/write access to the device registry.

#### Exit criteria

* You have an IoT Hub provisioned in your Azure subscription.

* You have properly selected the connection strings having appropriate permissions.

### Task 2: Configure the Smart Meter Simulator

If you want to save this connection string with your project (in case you stop debugging or otherwise close the simulator), you can set this as the default text for the text box. Follow these steps to configure the connection string:

#### Tasks to complete

* Edit the Fabrikam Smart Meter Simulator so that the IoT connection string text box has a value of your connection string to the IoT Hub having full permissions.

#### Exit criteria

* Your connection string should now be present every time you run the Smart Meter Simulator (in subsequent steps).

## Exercise 3: Completing the Smart Meter Simulator

Duration: 60 minutes

Fabrikam has left you a partially completed sample in the form of the Smart Meter Simulator solution. You will need to complete the missing lines of code that deal with device registration management and device telemetry transmission that communicate with your IoT Hub.

### Task 1: Implement device management with the IoT Hub

#### Tasks to complete

* In the solution, open **DeviceManager.cs**, and complete the lines of code below each of the TODO comments.

#### Exit criteria

17. There are sixteen TODOs in this file, and you should have completed all of them. You can use the Task List to see all the tasks at a glance. From the **View** menu, click **Task List**. There you will see a list of TODO tasks, where each task represents one line of code that needs to be completed.

### Task 2: Implement the communication of telemetry with the IoT Hub

#### Tasks to complete

* In the solution, open **Sensor.cs**, and complete the lines of code below each of the TODO comments.

#### Exit criteria

* There are four TODOs in this file, and you should have completed all of them. You can use the Task List to see all the tasks at a glance. From the **View** menu, click **Task List**. There you will see a list of TODO tasks, where each task represents one line of code that needs to be completed.

### Task 3: Verify device registration and telemetry

#### Tasks to complete

* Build and run the Smart Meter Simulator.

* Register all devices.

* By clicking on 1--10 of the windows within the building in the app, select the devices to install (they should turn yellow) and then activate them (after which they turn green).

* Use Device Explorer in the IoT blade to view the list of registered devices. How many of them have been activated in the list? How can you tell?

* Connect the activated devices and observe that they are sending telemetry.

#### Exit criteria

* You should have the Smart Meter Simulator running and actively transmitting telemetry.

## Exercise 4: Hot path data processing with Stream Analytics

Duration: 45 minutes

Fabrikam would like to visualize the "hot" data showing the average temperature reported by each device over a 5-minute window in Power BI.

### Task 1: Create a Stream Analytics job for hot path processing to Power BI

#### Tasks to complete

* Create an Azure Stream Analytics job that reads the JSON/UTF8 serialized telemetry from your IoT Hub and writes to Power BI.

* Query the input data over a 5-minute tumbling window.

#### Exit criteria

* Verify that your Stream Analytics job is receiving and processing telemetry from your Smart Meter Simulator instance.

### Task 2: Visualize hot data with Power BI

#### Tasks to complete

* Using Power BI create a report that contains a Column Chart visualization that on the x-axis has the device IDs and on the y-axis has the maximum value of the average temperature reported.

* Add another Column Chart visualization that plots the minimum value of the average temperature by device.

* Add a Table visualization that lists a table with a device ID and an average of the temperature columns.

#### Exit criteria

* You can view the report in Reading View and click one device data point to highlight it across all three of the visualizations.

## Exercise 5: Cold path data processing with HDInsight Spark

Duration: 60 minutes

Fabrikam would like to be able to capture all the "cold" data into scalable storage so that they can summarize it periodically using a Spark SQL query.

### Task 1: Create the Stream Analytics job for cold path processing

To capture all metrics for the cold path, set up another Stream Analytics job that will write all events to Blob storage for analyses by Spark running on HDInsight.

#### Tasks to complete

* Create an Azure Stream Analytics job that reads from your IoT Hub the JSON/UTF8 serialized telemetry and writes to Azure Storage blobs as CSV files.

* Query the input data so that all data points are written raw to storage without any filtering or summarization.

#### Exit criteria

* Verify that your Stream Analytics job is receiving and processing telemetry from your Smart Meter Simulator instance.

### Task 2: Verify CSV files in Blob storage

#### Tasks to complete

* Locate the CSV file created in Blob storage.

#### Exit criteria

* You have taken note of the container relative path to the CSV file as it appears in Blob storage.

### Task 3: Update pandas version on the Spark cluster

In this task, you will connect SSH into your HDInsight cluster, and update the version of pandas that Jupyter Notebook uses. This task is to address errors that are displayed because the autovizwidget in Jupyter needs a later version of pandas that has the API module.

#### Tasks to complete

* Create an SSH connection to the HDInsight Spark cluster.

* Execute the following command on the cluster to install the latest version of pandas.

    1.  sudo -HE /usr/bin/anaconda/bin/conda install pandas

#### Exit criteria

* The latest version of pandas is installed on the HDInsight Spark cluster.

### Task 4: Process with Spark SQL 

#### Tasks to complete

* Process the data by using a Jupyter notebook on HDInsight Spark to summarize the data by device ID, count of events per device, and the average temperature per device.

#### Exit criteria

* You can visualize the results of the query in the Jupyter notebook using a column chart that displays the ID as the x-axis and the average of the averageTemp as the y-axis.

## Exercise 6: Reporting device outages with IoT Hub Operations Monitoring

Duration: 20 minutes

Fabrikam would like to be alerted when devices disconnect and fail to reconnect after a period. Since they are already using PowerBI to visualize hot data, they would like to see a list of any of these devices in a report.

### Task 1: Enable verbose connection monitoring on the IoT Hub

To keep track of device connects and disconnects, we first need to enable verbose connection monitoring.

#### Tasks to complete

* Enable verbose connection monitoring.

#### Exit criteria

* You have enabled connection monitoring via IoT Hub Operations Monitoring, collecting all device connect and disconnect events.

### Task 2: Collect device connection telemetry with the hot path Stream Analytics job

Now that the device connections are being logged, update your hot path Stream Analytics job (the first one you created) with a new input that ingests device telemetry from Operations Monitoring. Next, create a query that joins all connected and disconnected events with a DATEDIFF function that only returns devices with a disconnect event, but no reconnect event within 120 seconds. Output the events to Power BI.

#### Tasks to complete

* Update your hot path Stream Analytics job (the first one you created) with a new input that ingests device telemetry from Operations Monitoring.

* Create a query that joins all connected and disconnected events with a DATEDIFF function that only returns devices with a disconnect event, but no reconnect event within 120 seconds.

* Output the events to Power BI.

#### Exit criteria

* Verify that your Stream Analytics job is still receiving and processing telemetry from your Smart Meter Simulator instance, and that new events are being captured after the devices have been disconnected for at least 2 minutes.

### Task 3: Test the device outage notifications

Register and activate a few devices on the Smart Meter Simulator, then connect them. Deactivate them without reconnecting in order for them to show up in the device outage report we will create in the next task.

#### Tasks to complete

* Register and activate a few devices on the Smart Meter Simulator, then connect them.

* Deactivate the devices without reconnecting, allowing them to show up in the device outage report we will create in the next task.

#### Exit criteria

* Devices have been connected, and unregistered for more than 120 seconds, so they will appear on the outage report.

### Task 4: Visualize disconnected devices with Power BI

#### Tasks to complete

* Create a Table visualization in Power BI, referencing the new device outage dataset created by the Stream Analytics output that was configured in Task 2.

#### Exit criteria

* Devices that connected, then were disconnected for longer than 120 seconds via the Smart Meter Simulator should be listed in the Table visualization in Power BI. Use the column headings to sort the devices by Device Id or Timestamp.

## After the hands-on lab

Duration: 10 mins

In this exercise, you will delete any Azure resources that were created in support of the lab. You should follow all steps provided after attending the Hands-on lab to ensure your account does not continue to be charged for lab resources.

### Task 1: Delete the resource group

1. Using the [Azure portal](https://portal.azure.com), navigate to the Resource group you used throughout this hands-on lab by selecting Resource groups in the left menu.
2. Search for the name of your research group, and select it from the list.
3. Select Delete in the command bar, and confirm the deletion by re-typing the Resource group name, and selecting Delete.

*You should follow all steps provided after attending the Hands-on lab.*