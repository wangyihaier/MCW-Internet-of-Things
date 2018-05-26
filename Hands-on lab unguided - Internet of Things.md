[](images/HeaderPic.png "Microsoft Cloud Workshops")

# Internet of Things

## Hands-on lab unguided

## March 2018

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.
© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

## Contents

<!-- TOC -->

- [Internet of Things](#internet-of-things)
    - [Hands-on lab unguided](#hands-on-lab-unguided)
    - [March 2018](#march-2018)
    - [Contents](#contents)
- [Internet of Things hands-on lab unguided](#internet-of-things-hands-on-lab-unguided)
    - [Abstract and learning objectives](#abstract-and-learning-objectives)
    - [Overview](#overview)
    - [Requirements](#requirements)
    - [Before the hands-on lab](#before-the-hands-on-lab)
        - [Task 1: Provision Power BI](#task-1--provision-power-bi)
        - [Task 2: Provision an HDInsight with Spark Cluster](#task-2--provision-an-hdinsight-with-spark-cluster)
        - [Task 3: Setup a lab virtual machine (VM)](#task-3--setup-a-lab-virtual-machine-vm)
        - [Task 4: Connect to the lab VM](#task-4--connect-to-the-lab-vm)
        - [Task 5: Prepare an SSH client](#task-5--prepare-an-ssh-client)
    - [Exercise 1: Environment setup](#exercise-1--environment-setup)
        - [Task 1: Download and open the Smart Meter Simulator project](#task-1--download-and-open-the-smart-meter-simulator-project)
    - [Exercise 2: IoT Hub provisioning](#exercise-2--iot-hub-provisioning)
        - [Task 1: Provision an IoT Hub](#task-1--provision-an-iot-hub)
        - [Task 2: Configure the Smart Meter Simulator](#task-2--configure-the-smart-meter-simulator)
    - [Exercise 3: Completing the Smart Meter Simulator](#exercise-3--completing-the-smart-meter-simulator)
        - [Task 1: Implement device management with the IoT Hub](#task-1--implement-device-management-with-the-iot-hub)
        - [Task 2: Implement the communication of telemetry with the IoT Hub](#task-2--implement-the-communication-of-telemetry-with-the-iot-hub)
        - [Task 3: Verify device registration and telemetry](#task-3--verify-device-registration-and-telemetry)
    - [Exercise 4: Hot path data processing with Stream Analytics](#exercise-4--hot-path-data-processing-with-stream-analytics)
        - [Task 1: Create a Stream Analytics job for hot path processing to Power BI](#task-1--create-a-stream-analytics-job-for-hot-path-processing-to-power-bi)
        - [Task 2: Visualize hot data with Power BI](#task-2--visualize-hot-data-with-power-bi)
    - [Exercise 5: Cold path data processing with HDInsight Spark](#exercise-5--cold-path-data-processing-with-hdinsight-spark)
        - [Task 1: Create the Stream Analytics job for cold path processing](#task-1--create-the-stream-analytics-job-for-cold-path-processing)
        - [Task 2: Verify CSV files in Blob storage](#task-2--verify-csv-files-in-blob-storage)
        - [Task 3: Update pandas version on the Spark cluster](#task-3--update-pandas-version-on-the-spark-cluster)
        - [Task 4: Process with Spark SQL](#task-4--process-with-spark-sql)
    - [Exercise 6: Reporting device outages with IoT Hub Operations Monitoring](#exercise-6--reporting-device-outages-with-iot-hub-operations-monitoring)
        - [Task 1: Enable verbose connection monitoring on the IoT Hub](#task-1--enable-verbose-connection-monitoring-on-the-iot-hub)
        - [Task 2: Collect device connection telemetry with the hot path Stream Analytics job](#task-2--collect-device-connection-telemetry-with-the-hot-path-stream-analytics-job)
        - [Task 3: Test the device outage notifications](#task-3--test-the-device-outage-notifications)
        - [Task 4: Visualize disconnected devices with Power BI](#task-4--visualize-disconnected-devices-with-power-bi)
    - [After the Hands-on Lab](#after-the-hands-on-lab)
        - [Task 1: Delete the resource group](#task-1--delete-the-resource-group)

<!-- /TOC -->

# Internet of Things hands-on lab unguided 

## Abstract and learning objectives

This workshop is designed to guide you through an implementation of an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. In this session, you will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path. After completing the package, you will be better able to implement device registration with the IoT Hub registry and visualize hot data with Power BI.

Learning objectives:

-   Implement a simulator sending telemetry from smart meters

-   Capture and process both hot and cold data using Stream Analytics and HDInsight with Spark

-   Visualize hot data with Power BI

## Overview

Fabrikam provides services and smart meters for enterprise energy (electrical power) management. Their "*You-Left-The-Light-On*" service enables the enterprise to understand their energy consumption.

In this hands-on lab, you will construct an end-to-end solution for an IoT scenario that includes device management; telemetry ingest; hot and cold path processing; and reporting.

## Requirements

1.  Microsoft Azure subscription must be pay-as-you-go or MSDN.

    -   Trial subscriptions will *not* work.

2.  A virtual machine configured with:

    -   Visual Studio Community 2017 or later

    -   Azure SDK 2.9 or later (Included with Visual Studio 2017)

3.  A running HDInsight Spark cluster (see [Before the Hands-on Lab](#_Before_the_Hands-on)).

## Before the hands-on lab

Duration: 45 minutes

In this exercise, you will set up your environment for use in the rest of the hands-on lab. You should follow all the steps provided in the Before the hands-on lab section to prepare your environment *before* attending the hands-on lab.

### Task 1: Provision Power BI

If you do not already have a Power BI account:

1.  Go to <https://powerbi.microsoft.com/features/>.

2.  Scroll down until you see the **Try Power BI for free!** section of the page and click the **Try Free\>** button.
    
    ![Screenshot of the Try Power BI Pro for free page.](images/Hands-onlabunguided-InternetofThingsimages/media/image2.png "Try Power BI Pro for Free ")

3.  On the page, enter your work email address (which should be the same account as the one you use for your Azure subscription), and select **Sign up**.
    
    ![The Get started page has a place to enter your work email address, and a sign up arrow.](images/Hands-onlabunguided-InternetofThingsimages/media/image3.png "Get started page")

4.  Follow the on-screen prompts, and your Power BI environment should be ready within minutes. You can always return to it via <https://app.powerbi.com/>.

### Task 2: Provision an HDInsight with Spark Cluster

Using the Azure Portal, provision a new HDInsight cluster.

1.  Open a browser, and go to the Azure portal (<https://portal.azure.com>).

2.  Select **+New**, select **Data + Analytics**, **HDInsight**.
    
    ![The Azure Portal has the New button selected in the left pane. In the New pane, under Azure Marketplace, Data + Analytics is selected, and under Featured, HDInsights is selected.](images/Hands-onlabunguided-InternetofThingsimages/media/image4.png "Azure Portal, New pane")

3.  On the HDInsight blade, select **Custom (size, settings, apps)**.
    
    ![The HDInsight blade has the Custom (size, settings, apps) selected.](images/Hands-onlabunguided-InternetofThingsimages/media/image5.png "HDInsight blade")

4.  On the Basics blade, enter the following settings:

    -   Cluster name: Enter a unique name (verified by the green checkmark).

    -   Subscription: Select the Azure subscription into which you want to deploy the cluster.

    -   Custer type: Select ***Configure required settings***.

        ![Screenshot of the Configure required settings link.](images/Hands-onlabunguided-InternetofThingsimages/media/image6.png "Configure required settings link")

        i.  On the Cluster configuration blade, set the **Cluster type** to **Spark** and the **Version** to **Spark 2.1.0 (HDI 3.6)**. Note that the Operating System option for the Spark cluster is fixed to Linux. 
        
        ![The Cluster configuration has the cluster type and version selected. The cluster type is Spark. The Spark version is 2.1.0 (HDI 3.6)](images/Hands-onlabunguided-InternetofThingsimages/media/image7.png "Cluster configuration")

        ii. Select **Select** to close the Cluster configuration blade.

    -   Cluster login username: Leave as **admin**.

    -   Cluster login password: Enter **Password.1!!** for the admin password.

    -   Secure Shell (SSH) username: Enter **sshuser**.

    -   Use same password as cluster login: Ensure the checkbox is **checked**.

    -   Resource group: Select the Create new radio button, and enter **iot-hol** for the resource group name.

    -   Location: Select the desired location from the dropdown list, and remember this, as the same location will be used for all other Azure resources.
        
        ![The Basics blade fields display the previously mentioned settings.](images/Hands-onlabunguided-InternetofThingsimages/media/image8.png "Basics blade")

    -   Select **Next** to move on to the storage settings.

5.  On the Storage blade:

    -   Primary storage type: Leave set to **Azure Storage.**

    -   Selection Method: Leave set to **My subscriptions**.

    -   Select a Storage account: Select **Create new**, and enter a name for the storage account, such as iotholstorage.

    -   Default container: Enter **iotcontainer**.

    -   Additional storage accounts: Leave unconfigured.

    -   Data Lake Store access: Leave unconfigured.

    -   Metastore Settings: Leave blank.
        
        ![The Storage blade fields display the previously mentioned settings.](images/Hands-onlabunguided-InternetofThingsimages/media/image9.png "Storage blade")

    -   Select **Next**.

6.  Select **Next** on the Applications (optional) blade. No applications are being added.

7. On the Cluster size blade:

    -   Number of worker nodes: Leave set to **4**.

    -   Select **Worker node size**, and select **D12 v2**, then select **Select**.
        
        ![The Cluster size blade, worker node size section has the D12 V2 Standard option circled.](images/Hands-onlabunguided-InternetofThingsimages/media/image10.png "Cluster size blade, worker node size section")

    -   Leave **Head node size**, set to the default, **D12 v2**.
        
        ![The Cluster size blade fields display the previously mentioned settings.](images/Hands-onlabunguided-InternetofThingsimages/media/image11.png "Cluster Size Blade")

    -   Select **Next**.

8. Select **Next** on the Advanced settings blade to move to the Cluster summary blade.

9. Select **Create** on the Cluster summary blade to create the cluster.

10. It will take approximately 20 minutes to create your cluster. You can move on to the steps below while the cluster is provisioning.

### Task 3: Setup a lab virtual machine (VM)

1.  In the [Azure Portal](https://portal.azure.com/), select **+New**, then type "Visual Studio" into the search bar. Select **Visual Studio Community 2017 (latest release) on Windows Server 2016 (x64)** from the results**.
    \
    ![In the Azure Portal, Everything pane, Visual studio is typed in the search field. Under Results, under Name, Visual Studio Community 2017 is circled.](images/Hands-onlabunguided-InternetofThingsimages/media/image12.png "Azure Portal, Everything pane")

2.  On the blade that comes up, at the bottom, ensure the deployment model is set to **Resource Manager** and select **Create**.

    ![Under Select a deployment model, Resource Manager is selected.](images/Hands-onlabunguided-InternetofThingsimages/media/image13.png "Resource Manager option")

3.  Set the following configuration on the Basics tab.

-   Name: Enter **LabVM**.

-   VM disk type: Select **SSD**.

-   User name: Enter **demouser**

-   Password: Enter **Password.1!!**

-   Subscription: Select the same subscription you used to create your cluster in [Task 1](#task-1-provision-power-bi).

-   Resource Group: Select Use existing, and select the resource group you provisioned while creating your cluster in Task 1.

-   Location: Select the same region you used in Task 1 while creating your cluster.
    
    ![The Basics blade fields display with the previously mentioned settings.](images/Hands-onlabunguided-InternetofThingsimages/media/image14.png "Basics blade")

4.  Select **OK** to move to the next step.

5.  On the Choose a size blade, ensure the Supported disk type is set to SSD, and select View all. This machine won't be doing much heavy lifting, so selecting **DS2\_V3** **Standard** is a good baseline option.
    
    ![The Choose a size blade, worker node size section has D2S\_V3 Standard circled. Supported disk type is set to SSD, and the View all button is circled.](images/Hands-onlabunguided-InternetofThingsimages/media/image15.png "Choose a size blade, worker node size section")

6.  Select **Select** to move on to the Settings blade.

7.  Accept all the default values on the Settings blade, and Select **OK**.

8.  Select **Create** on the Create blade to provision the virtual machine.
    
    ![Screenshot of the Create blade, Summary section, showing that Validation passed, and detailing the offer details.](images/Hands-onlabunguided-InternetofThingsimages/media/image16.png "Create blade, Summary section")

9.  It may take 10+ minutes for the virtual machine to complete provisioning.

### Task 4: Connect to the lab VM

1.  Connect to the Lab VM. (If you are already connected to your Lab VM, skip to Step 9.)

2.  From the left side menu in the Azure portal, click on **Resource groups**, then enter your resource group name into the filter box, and select it from the list.
    
    ![In the Azure Portal, Resource groups is circled in the menu on the left. In the Resource groups pane, iot displays in the Subscriptions search field. Under Name, iot-hol is circled.](images/Hands-onlabunguided-InternetofThingsimages/media/image17.png "Azure Portal, Resource groups pane")

3.  Next, select your lab virtual machine, **LabVM**, from the list.
    
    ![Under Name, LabVM is selected.](images/Hands-onlabunguided-InternetofThingsimages/media/image18.png "LabVM option")

4.  On your Lab VM blade, select **Connect** from the top menu.
    
    ![The Connect button is circled on the Lab VM blade menu bar.](images/Hands-onlabunguided-InternetofThingsimages/media/image19.png "Lab VM blade menu bar")

5.  Download and open the RDP file.

6.  Select **Connect** on the Remote Desktop Connection dialog.
    
    ![The Connect button is circled on the Remote Desktop Connection dialog box, which asks if you still want to connect, even though the publisher of the remote connection can\'t be identified.](images/Hands-onlabunguided-InternetofThingsimages/media/image20.png "Remote Desktop Connection dialog box")

7.  Enter the following credentials (or the non-default credentials if you changed them):

    -   User name: **demouser**

    -   Password: **Password.1!!

        ![Screenshot of the Windows Security, Enter your credentials window for demouser.](images/Hands-onlabunguided-InternetofThingsimages/media/image21.png "Windows Security, Enter your credentials window")

8.  Select **Yes** to connect, if prompted that the identity of the remote computer cannot be verified.
    
    ![The Yes button is circled on the Remote Desktop Connection dialog box, which asks if you still want to connect, even though the identity of the remote connection can\'t be identified.](images/Hands-onlabunguided-InternetofThingsimages/media/image22.png "Remote Desktop Connection dialog box")

9.  Once logged in, launch the Server Manager. This should start automatically, but you can access it via the Start menu if it does not start.

10. Select **Local Server**, then select **On** next to IE Enhanced Security Configuration.
    
    ![In Server Manager, in the left pane, Local Server is selected. In the right, Properties pane, a callout points to On, next to IE Enhanced Security Configuration.](images/Hands-onlabunguided-InternetofThingsimages/media/image23.png "Server Manager")

11. In the Internet Explorer Enhanced Security Configuration dialog, select **Off under Administrators**, then select **OK**.
    
    ![On the Internet Explorer Enhanced Security Configuration dialog box, under Adminstrators, the Off radio button is selected and circled.](images/Hands-onlabunguided-InternetofThingsimages/media/image24.png "Internet Explorer Enhanced Security Configuration dialog box")

12. Close the Server Manager.

### Task 5: Prepare an SSH client

In this task, you will download, install, and prepare the Git Bash SSH client that you will use to access your HDInsight cluster from your Lab VM.

1.  On your Lab VM, open a browser, and navigate to <https://git-scm.com/downloads> to download Git Bash.
    
    ![Screenshot of the Git Bash Downloads webpage.](images/Hands-onlabunguided-InternetofThingsimages/media/image25.png "Download Git Bash")

2.  Select the download for your OS, and then select the Download 2.15.x for... button.

3.  Run the downloaded installer, select Next on each screen to accept the defaults.

4.  On the last screen, select Install to complete the installation.
    
    ![The Git Setup, Configuring extra options page has two checkboxes selected, for Enable file system caching, and Enable Git Credential Manager. ](images/Hands-onlabunguided-InternetofThingsimages/media/image26.png "Git Setup Wizard Configuring options page")

<!-- -->

1. When the install is complete, you will be presented with the following screen:
    
    ![The Completing the Git Setup Wizard page has the check box selected to Launch Git Bash.](images/Hands-onlabunguided-InternetofThingsimages/media/image27.png "Git Setup Wizard, Completing setup page")

15. Check the **Launch Git Bash checkbox**, and uncheck **View Release Notes**. Select **Finish**.

16. Leave the bash window open, as you will use it later in this lab.

## Exercise 1: Environment setup

Duration: 10 minutes

Fabrikam has provided a Smart Meter Simulator that they use to simulate device registration, as well as the generation and transmission of telemetry data. They have asked you to use this as the starting point for integrating their smart meters with Azure.

### Task 1: Download and open the Smart Meter Simulator project

*Tasks to complete*:

-   From your Lab VM, download the Smart Meter Simulator starter project from the following URL: <https://bit.ly/2wMSwsH> (Note: the URL is case-sensitive).

-   Unzip the contents to the folder C:\\SmartMeter\\.

-   Open SmartMeterSimulator.sln with Visual Studio 2017.

*Exit criteria*:

-   The SmartMeterSimulation solution is open in Visual Studio on your Lab VM.

**Note:** If you attempt to build the solution at this point, you will see many build errors. This is intentional. You will correct these in the exercises that follow.

## Exercise 2: IoT Hub provisioning

Duration: 20 minutes

In your architecture design session with Fabrikam, it was agreed that you would use an Azure IoT Hub to manage both the device registration and telemetry ingest from the Smart Meter Simulator. Your team also identified the Microsoft provided Device Explorer project that Fabrikam can use to view the list and status of devices in the IoT Hub registry.

### Task 1: Provision an IoT Hub

In these steps, you will provision an instance of IoT Hub.

*Tasks to complete*:

-   Provision an IoT Hub instance.

-   Determine and take note of the connection strings required for 1) full control and 2) read/write access to the device registry.

*Exit criteria*:

-   You have an IoT Hub provisioned in your Azure subscription.

-   You have properly selected the connection strings having appropriate permissions.

### Task 2: Configure the Smart Meter Simulator

If you want to save this connection string with your project (in case you stop debugging or otherwise close the simulator), you can set this as the default text for the text box. Follow these steps to configure the connection string:

*Tasks to complete*:

-   Edit the Fabrikam Smart Meter Simulator so that the IoT connection string text box has a value of your connection string to the IoT Hub having full permissions.

*Exit criteria*:

-   Your connection string should now be present every time you run the Smart Meter Simulator (in subsequent steps).

## Exercise 3: Completing the Smart Meter Simulator

Duration: 60 minutes

Fabrikam has left you a partially completed sample in the form of the Smart Meter Simulator solution. You will need to complete the missing lines of code that deal with device registration management and device telemetry transmission that communicate with your IoT Hub.

### Task 1: Implement device management with the IoT Hub

*Tasks to complete*:

-   In the solution, open **DeviceManager.cs**, and complete the lines of code below each of the TODO comments.

*Exit criteria*:

17. There are sixteen TODOs in this file, and you should have completed all of them. You can use the Task List to see all the tasks at a glance. From the **View** menu, click **Task List**. There you will see a list of TODO tasks, where each task represents one line of code that needs to be completed.

### Task 2: Implement the communication of telemetry with the IoT Hub

*Tasks to complete*:

-   In the solution, open **Sensor.cs**, and complete the lines of code below each of the TODO comments.

*Exit criteria*:

-   There are four TODOs in this file, and you should have completed all of them. You can use the Task List to see all the tasks at a glance. From the **View** menu, click **Task List**. There you will see a list of TODO tasks, where each task represents one line of code that needs to be completed.

### Task 3: Verify device registration and telemetry

*Tasks to complete*:

-   Build and run the Smart Meter Simulator.

-   Register all devices.

-   By clicking on 1--10 of the windows within the building in the app, select the devices to install (they should turn yellow) and then activate them (after which they turn green).

-   Use Device Explorer in the IoT blade to view the list of registered devices. How many of them have been activated in the list? How can you tell?

-   Connect the activated devices and observe that they are sending telemetry.

*Exit criteria*:

-   You should have the Smart Meter Simulator running and actively transmitting telemetry.

## Exercise 4: Hot path data processing with Stream Analytics

Duration: 45 minutes

Fabrikam would like to visualize the "hot" data showing the average temperature reported by each device over a 5-minute window in Power BI.

### Task 1: Create a Stream Analytics job for hot path processing to Power BI

*Tasks to complete*:

-   Create an Azure Stream Analytics job that reads the JSON/UTF8 serialized telemetry from your IoT Hub and writes to Power BI.

-   Query the input data over a 5-minute tumbling window.

*Exit criteria*:

-   Verify that your Stream Analytics job is receiving and processing telemetry from your Smart Meter Simulator instance.

### Task 2: Visualize hot data with Power BI

*Tasks to complete*:

-   Using Power BI create a report that contains a Column Chart visualization that on the x-axis has the device IDs and on the y-axis has the maximum value of the average temperature reported.

-   Add another Column Chart visualization that plots the minimum value of the average temperature by device.

-   Add a Table visualization that lists a table with a device ID and an average of the temperature columns.

*Exit criteria*:

-   You can view the report in Reading View and click one device data point to highlight it across all three of the visualizations.

## Exercise 5: Cold path data processing with HDInsight Spark

Duration: 60 minutes

Fabrikam would like to be able to capture all the "cold" data into scalable storage so that they can summarize it periodically using a Spark SQL query.

### Task 1: Create the Stream Analytics job for cold path processing

To capture all metrics for the cold path, set up another Stream Analytics job that will write all events to Blob storage for analyses by Spark running on HDInsight.

*Tasks to complete*:

-   Create an Azure Stream Analytics job that reads from your IoT Hub the JSON/UTF8 serialized telemetry and writes to Azure Storage blobs as CSV files.

-   Query the input data so that all data points are written raw to storage without any filtering or summarization.

*Exit criteria*:

-   Verify that your Stream Analytics job is receiving and processing telemetry from your Smart Meter Simulator instance.

### Task 2: Verify CSV files in Blob storage

*Tasks to complete*:

-   Locate the CSV file created in Blob storage.

*Exit criteria*:

-   You have taken note of the container relative path to the CSV file as it appears in Blob storage.

### Task 3: Update pandas version on the Spark cluster

In this task, you will connect SSH into your HDInsight cluster, and update the version of pandas that Jupyter Notebook uses. This task is to address errors that are displayed because the autovizwidget in Jupyter needs a later version of pandas that has the API module.

*Tasks to complete*:

-   Create an SSH connection to the HDInsight Spark cluster.

-   Execute the following command on the cluster to install the latest version of pandas.

    1.  sudo -HE /usr/bin/anaconda/bin/conda install pandas

*Exit criteria*:

-   The latest version of pandas is installed on the HDInsight Spark cluster.

### Task 4: Process with Spark SQL 

*Tasks to complete*:

-   Process the data by using a Jupyter notebook on HDInsight Spark to summarize the data by device ID, count of events per device, and the average temperature per device.

*Exit criteria*:

-   You can visualize the results of the query in the Jupyter notebook using a column chart that displays the ID as the x-axis and the average of the averageTemp as the y-axis.

## Exercise 6: Reporting device outages with IoT Hub Operations Monitoring

Duration: 20 minutes

Fabrikam would like to be alerted when devices disconnect and fail to reconnect after a period. Since they are already using PowerBI to visualize hot data, they would like to see a list of any of these devices in a report.

### Task 1: Enable verbose connection monitoring on the IoT Hub

To keep track of device connects and disconnects, we first need to enable verbose connection monitoring.

*Tasks to complete*:

-   Enable verbose connection monitoring.

*Exit criteria*:

-   You have enabled connection monitoring via IoT Hub Operations Monitoring, collecting all device connect and disconnect events.

### Task 2: Collect device connection telemetry with the hot path Stream Analytics job

Now that the device connections are being logged, update your hot path Stream Analytics job (the first one you created) with a new input that ingests device telemetry from Operations Monitoring. Next, create a query that joins all connected and disconnected events with a DATEDIFF function that only returns devices with a disconnect event, but no reconnect event within 120 seconds. Output the events to Power BI.

*Tasks to complete*:

-   Update your hot path Stream Analytics job (the first one you created) with a new input that ingests device telemetry from Operations Monitoring.

-   Create a query that joins all connected and disconnected events with a DATEDIFF function that only returns devices with a disconnect event, but no reconnect event within 120 seconds.

-   Output the events to Power BI.

*Exit criteria*:

-   Verify that your Stream Analytics job is still receiving and processing telemetry from your Smart Meter Simulator instance, and that new events are being captured after the devices have been disconnected for at least 2 minutes.

### Task 3: Test the device outage notifications

Register and activate a few devices on the Smart Meter Simulator, then connect them. Deactivate them without reconnecting in order for them to show up in the device outage report we will create in the next task.

*Tasks to complete*:

-   Register and activate a few devices on the Smart Meter Simulator, then connect them.

-   Deactivate the devices without reconnecting, allowing them to show up in the device outage report we will create in the next task.

*Exit criteria*:

-   Devices have been connected, and unregistered for more than 120 seconds, so they will appear on the outage report.

### Task 4: Visualize disconnected devices with Power BI

*Tasks to complete*:

-   Create a Table visualization in Power BI, referencing the new device outage dataset created by the Stream Analytics output that was configured in Task 2.

*Exit criteria*:

-   Devices that connected, then were disconnected for longer than 120 seconds via the Smart Meter Simulator should be listed in the Table visualization in Power BI. Use the column headings to sort the devices by Device Id or Timestamp.

## After the Hands-on Lab 

Duration: 10 minutes

In this exercise, attendees will deprovision any Azure resources that were created in support of the lab.

### Task 1: Delete the resource group

*Tasks to complete*:

-   Delete the resource group you created for this hands-on lab.

*Exit criteria*:

-   All resources provisioned for this lab have been deleted, or you have stopped on the major, cost incurring services used in this hands-on lab.

