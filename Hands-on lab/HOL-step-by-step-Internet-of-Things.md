![Microsoft Cloud Workshop](../media/ms-cloud-workshop.png "Microsoft Cloud Workshop")

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.
© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

# Internet of Things hands-on lab step-by-step

Updated June 2018

In this hands-on lab, you will implement an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path. After completing the hands-on lab, you will have a better understanding of implementing device registration with the IoT Hub Device Provisioning Service and visualizing hot data with Power BI.

## Contents

* [Abstract](#abstract)
* [Overview](#overview)
* [Solution architecture](#solution-architecture)
* [Requirements](#requirements)
* [Exercise 1: IoT Hub provisioning](#exercise-1-iot-hub-provisioning)
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

## Exercise 1: IoT Hub provisioning

Duration: 15 minutes

In your architecture design session with Fabrikam, it was agreed that you would use an Azure IoT Hub to manage both the device registration and telemetry ingest from the Smart Meter Simulator. Your team also identified the Microsoft provided Device Explorer project that Fabrikam can use to view the list and status of devices in the IoT Hub registry.

### Task 1: Provision IoT Hub

In these steps, you will provision an instance of IoT Hub.

1. In your browser, navigate to the [Azure portal](https://portal.azure.com), select **+Create a resource** in the navigation pane, enter "iot" into the Search the Marketplace box, select **IoT Hub** from the results, and select **Create**.

    ![+Create a resource is highlighted in the navigation page of the Azure portal, and "iot" is entered into the Search the Marketplace box. IoT Hub is highlighted in the search results.](./media/create-resource-iot-hub.png "Create an IoT Hub")

2. On the IoT Hub blade Basics tab, enter the following:

    * **Subscription**: Select the subscription you are using for this hands-on lab

    * **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group

    * **Region**: Select the location you are using for this hands-on lab

    * **IoT Hub Name**: Enter a unique name, such as smartmeter-hub-SUFFIX

        ![The Basics blade for IoT Hub is displayed, with the values specified above entered into the appropriate fields.](./media/iot-hub-basics-blade.png "Create IoT Hub Basic blade")

    * Select **Next: Size and Scale**.

    * On the Size and scale blade, accept the default Pricing and scale tier of S1: Standard tier, and select **Review + create**.

    * Select **Create** on the Review + create blade.

3. When the IoT Hub deployment is completed, you will receive a notification in the Azure portal. Select **Go to resource** in the notification.

    ![Screenshot of the Deployment succeeded message, with the Go to resource button highlighted.](./media/iot-hub-deployment-succeeded.png "Deployment succeeded message")

4. From the IoT Hub's Overview blade, select **Shared access policies** under Settings on the left-hand menu.

    ![Screenshot of the Overview blade, settings section. Under Settings, Shared access policies is highlighted.](./media/iot-hub-shared-access-policies.png "Overview blade, settings section")

5. Select **iothubowner** policy.

    ![The Azure portal is shown with the iothubowner selected.](./media/iot-hub-shared-access-policies-iothubowner.png "IoT Hub Owner shared access policy")

6. In the **iothubowner** blade, select the Copy button to the right of the **Connection string - primary key** field. You will be pasting the connection string value into a TextBox's Text property value in the next task.

    ![Screenshot of the iothubowner blade. The connection string - primary key field is highlighted.](./media/iot-hub-shared-access-policies-iothubowner-blade.png "iothubowner blade")

### Task 2: Configure the Smart Meter Simulator

If you want to save this connection string with your project (in case you stop debugging or otherwise close the simulator), you can set this as the default text for the text box. Follow these steps to configure the connection string:

1. Return to the `SmartMeterSimulator` solution in Visual Studio on your Lab VM.

2. In the Solution Explorer, double-click `MainForm.cs` to open it. (If the Solution Explorer is not in the upper left corner of your Visual Studio instance, you can find it under the View menu in Visual Studio.)

    ![In the Visual Studio Solution Explorer window, SmartMeterSimulator is expanded, and under it, MainForm.cs is highlighted.](media/visual-studio-solution-explorer-mainform-cs.png "Visual Studio Solution Explorer")

3. In the Windows Forms designer surface, click the **IoT Hub Connection String TextBox** to select it.

    ![The Windows Form designer surface is opened to the MainForm.cs tab. The IoT Hub Connection String is highlighted, but is empty.](./media/smart-meter-simulator-iot-hub-connection-string.png "Windows Form designer surface")

4. In the Properties panel, scroll until you see the **Text** property. Paste your IoT Hub connection string value copied in step 7 of the previous task into the value for the Text property. (If the properties window is not visible below the Solution Explorer, right-click the TextBox, and select **Properties**.)

    ![In the Properties panel, the Text property is highlighted, and is set to HostName=smartmeter-hub.](./media/smart-meter-simulator-iot-hub-connection-string-text-property.png "Solution Explorer")

5. Your connection string should now be present every time you run the Smart Meter Simulator.

    ![The Windows Form designer surface is opened to the MainForm.cs tab. The IoT Hub Connection String now displays.](./media/smart-meter-simulator-iot-hub-connection-string-populated.png "IoT Hub Connection String dialog")

6. Save `MainForm.cs`.

## Exercise 2: Completing the Smart Meter Simulator

Duration: 60 minutes

Fabrikam has left you a partially completed sample in the form of the Smart Meter Simulator solution. You will need to complete the missing lines of code that deal with device registration management and device telemetry transmission that communicate with your IoT Hub.

### Task 1: Implement device management with the IoT Hub

1. In Visual Studio on your Lab VM, use Solution Explorer to open the file `DeviceManager.cs`.

2. From the Visual Studio **View** menu, click **Task List**.

    ![On the Visual Studio View menu, Task List is selected.](media/visual-studio-view-menu-task-list.png "Visual Studio View menu")

3. In the Task List, you will see a list of `TODO` tasks, where each task represents one line of code that needs to be completed. Complete the line of code below each `TODO` using the code below as a reference.

4. The following code represents the completed tasks in `DeviceManager.cs`:

    ```csharp
    class DeviceManager
    {
        static string connectionString;
        static RegistryManager registryManager;

        public static string HostName { get; set; }

        public static void IotHubConnect(string cnString)
        {
            connectionString = cnString;

            //TODO: 1.Create an instance of RegistryManager from connectionString
            registryManager = RegistryManager.CreateFromConnectionString(connectionString);

            var builder = IotHubConnectionStringBuilder.Create(cnString);

            HostName = builder.HostName;
        }

        /// <summary>
        /// Register a single device with the IoT hub. The device is initially registered in a
        /// disabled state.
        /// </summary>
        /// <param name="connectionString"></param>
        /// <param name="deviceId"></param>
        /// <returns></returns>
        public async static Task<string> RegisterDevicesAsync(string connectionString, string deviceId)
        {
            //Make sure we're connected
            if (registryManager == null)
                IotHubConnect(connectionString);

            //TODO: 2.Create new device
            Device device = new Device(deviceId);

            //TODO: 3.Initialize device with a status of Disabled
            //Enabled in a subsequent step
            device.Status = DeviceStatus.Disabled;

            try
            {
                //TODO: 4.Register the new device
                device = await registryManager.AddDeviceAsync(device);
            }
            catch (Exception ex)
            {
                if (ex is DeviceAlreadyExistsException ||
                    ex.Message.Contains("DeviceAlreadyExists"))
                {
                    //TODO: 5.Device already exists, get the registered device
                    device = await registryManager.GetDeviceAsync(deviceId);

                    //TODO: 6.Ensure the device is disabled until Activated later
                    device.Status = DeviceStatus.Disabled;

                    //TODO: 7.Update IoT Hubs with the device status change
                    await registryManager.UpdateDeviceAsync(device);
                }
                else
                {
                    MessageBox.Show($"An error occurred while registering one or more devices:\r\n{ex.Message}");
                }
            }

            //return the device key
            return device.Authentication.SymmetricKey.PrimaryKey;
        }

        /// <summary>
        /// Activate an already registered device by changing its status to Enabled.
        /// </summary>
        /// <param name="connectionString"></param>
        /// <param name="deviceId"></param>
        /// <param name="deviceKey"></param>
        /// <returns></returns>
        public async static Task<bool> ActivateDeviceAsync(string connectionString, string deviceId, string deviceKey)
        {
            //Server-side management function to enable the provisioned device
            //to connect to IoT Hub after it has been installed locally.
            //If device id device key are valid, Activate (enable) the device.

            //Make sure we're connected
            if (registryManager == null)
                IotHubConnect(connectionString);

            bool success = false;
            Device device;

            try
            {
                //TODO: 8.Fetch the device
                device = await registryManager.GetDeviceAsync(deviceId);

                //TODO: 9.Verify the device keys match
                if (deviceKey == device.Authentication.SymmetricKey.PrimaryKey)
                {
                    //TODO: 10.Enable the device
                    device.Status = DeviceStatus.Enabled;

                    //TODO: 11.Update IoT Hubs
                    await registryManager.UpdateDeviceAsync(device);

                    success = true;
                }
            }
            catch(Exception)
            {
                success = false;
            }

            return success;
        }

        /// <summary>
        /// Deactivate a single device in the IoT Hub registry.
        /// </summary>
        /// <param name="connectionString"></param>
        /// <param name="deviceId"></param>
        /// <returns></returns>
        public async static Task<bool> DeactivateDeviceAsync(string connectionString, string deviceId)
        {
            //Make sure we're connected
            if (registryManager == null)
                IotHubConnect(connectionString);

            bool success = false;
            Device device;

            try
            {
                //TODO: 12.Lookup the device from the registry by deviceId
                device = await registryManager.GetDeviceAsync(deviceId);

                //TODO: 13.Disable the device
                device.Status = DeviceStatus.Disabled;

                //TODO: 14.Update the registry
                await registryManager.UpdateDeviceAsync(device);

                success = true;
            }
            catch (Exception)
            {
                success = false;
            }

            return success;
        }

        /// <summary>
        /// Unregister a single device from the IoT Hub Registry
        /// </summary>
        /// <param name="connectionString"></param>
        /// <param name="deviceId"></param>
        /// <returns></returns>
        public async static Task UnregisterDevicesAsync(string connectionString, string deviceId)
        {
            //Make sure we're connected
            if (registryManager == null)
                IotHubConnect(connectionString);

                //TODO: 15.Remove the device from the Registry
                await registryManager.RemoveDeviceAsync(deviceId);
        }

        /// <summary>
        /// Unregister all the devices managed by the Smart Meter Simulator
        /// </summary>
        /// <param name="connectionString"></param>
        /// <returns></returns>
        public async static Task UnregisterAllDevicesAsync(string connectionString)
        {
            //Make sure we're connected
            if (registryManager == null)
               IotHubConnect(connectionString);

            for(int i = 0; i <= 9; i++)
            {
                string deviceId = "Device" + i.ToString();

                //TODO: 16.Remove the device from the Registry
                await registryManager.RemoveDeviceAsync(deviceId);
            }
        }
    }
    ```
5. Save `DeviceManager.cs`.

### Task 2: Implement the communication of telemetry with the IoT Hub

1. Open `Sensor.cs` from the Solution Explorer, and complete the `TODO` items indicated within the code that are responsible for transmitting telemetry data to the IoT Hub.

2. The following code shows the completed result:

    ```csharp
    class Sensor
    {
        private DeviceClient _DeviceClient;
        private string _IotHubUri { get; set; }
        public string DeviceId { get; set; }
        public string DeviceKey { get; set; }
        public DeviceState State { get; set; }
        public string StatusWindow { get; set; }
        public double CurrentTemperature
        {
            get
            {
                double avgTemperature = 70;
                Random rand = new Random();

                double currentTemperature = avgTemperature + rand.Next(-6, 6);

                if(currentTemperature <= 68)
                    TemperatureIndicator = SensorState.Cold;
                else if(currentTemperature > 68 && currentTemperature < 72)
                    TemperatureIndicator = SensorState.Normal;
                else if(currentTemperature >= 72)
                    TemperatureIndicator = SensorState.Hot;

                return currentTemperature;
            }
        }

        public SensorState TemperatureIndicator { get; set; }

        public Sensor(string iotHubUri, string deviceId, string deviceKey)
        {
            _IotHubUri = iotHubUri;
            DeviceId = deviceId;
            DeviceKey = deviceKey;
            State = DeviceState.Registered;
        }

        public void InstallDevice(string statusWindow)
        {
            StatusWindow = statusWindow;
            State = DeviceState.Installed;
        }

        /// <summary>
        /// Connect a device to the IoT Hub by instantiating a DeviceClient for that Device by Id and Key.
        /// </summary>
        public void ConnectDevice()
        {
            //TODO: 17. Connect the Device to Iot Hub by creating an instance of DeviceClient
            _DeviceClient = DeviceClient.Create(_IotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey(DeviceId, DeviceKey));

            //Set the Device State to Ready
            State = DeviceState.Ready;
        }

        public void DisconnectDevice()
        {
            //Delete the local device client
            _DeviceClient = null;

            //Set the Device State to Activate
            State = DeviceState.Activated;
        }

        /// <summary>
        /// Send a message to the IoT Hub from the Smart Meter device
        /// </summary>
        public async void SendMessageAsync()
        {
            var telemetryDataPoint = new
            {
                id = DeviceId,
                time = DateTime.UtcNow.ToString("o"),
                temp = CurrentTemperature
            };

            //TODO: 18.Serialize the telemetryDataPoint to JSON
            var messageString = JsonConvert.SerializeObject(telemetryDataPoint);

            //TODO: 19.Encode the JSON string to ASCII as bytes and create new Message with the bytes
            var message = new Message(Encoding.ASCII.GetBytes(messageString));

            //TODO: 20.Send the message to the IoT Hub
            var sendEventAsync = _DeviceClient?.SendEventAsync(message);
            if (sendEventAsync != null) await sendEventAsync;
        }
    }
    ```
3. Save `Sensor.cs`.

### Task 3: Verify device registration and telemetry

In this task, you will build and run the Smart Meter Simulator project.

1. In Visual Studio select Build from the Visual Studio menu, then select Build Solution.

2. Run the Smart Meter Simulator, by selecting the green Start button on the Visual Studio toolbar.

    ![The green Start button is highlighted on the Visual Studio toolbar.](media/visual-studio-toolbar-start.png "Visual Studio toolbar")

3. Select **Register** on the Smart Meter Simulator dialog, which should cause the windows within the building to change from black to gray.

    ![In addition to the IoT Hub Connection String, the Smart Meter Simulator has two buildings with 10 windows. The color of the windows indicating the status of the devices. Currently, all windows are gray.](media/smart-meter-simulator-register.png "Fabrikam Smart Meter Simulator")

4. Select a few of the windows. Each represents a device for which you want to simulate device installation. The selected windows should turn yellow.

    ![The Smart Meter Simulator now has three white windows, with the other seven remaining gray.](media/smart-meter-simulator-window-select.png "Fabrikam Smart Meter Simulator")

5. Select **Activate** to simulate changing the device status from disabled to enabled in the IoT Hub Registry. The selected windows should turn green.

    ![On the Smart Meter Simulator, the Activate button is highlighted, and the three white windows have now turned to green.](media/smart-meter-simulator-activate.png "Fabrikam Smart Meter Simulator")

6. At this point, you have registered 10 devices (the gray windows) but activated only the ones you selected (in green). To view this list of devices, you will switch over to the Azure Portal, and open the IoT Hub you provisioned.

7. From the IoT Hub blade, select **IoT Devices** under Explorers on the left-hand menu.

    ![On the IoT Hub blade, in the Explorers section, under Explorers, IoT Devices is highlighted.](media/iot-hub-explorers-iot-devices.png "IoT Hub blade, Explorers section")

8. You should see all 10 devices listed, with the ones that you activated having a status of **enabled**.

    ![Devices in the Device ID list have a status of either enabled or disabled.](media/iot-hub-iot-devices-list.png "Device ID list")

9. Return to the **Smart Meter Simulator** window.

10. Select **Connect**. Within a few moments, you should begin to see activity as the windows change color indicating the smart meters are transmitting telemetry. The grid on the left will list each telemetry message transmitted and the simulated temperature value.

    ![On the Smart Meter Simulator, the Connect button is highlighted, and one of the green windows has now turned to blue. The current windows count is now seven gray, two green, and one blue.](media/smart-meter-simulator-connect.png "Fabrikam Smart Meter Simulator")

11. Allow the smart meter to continue to run. (Whenever you want to stop the transmission of telemetry, select the **Disconnect** button.)

## Exercise 3: Hot path data processing with Stream Analytics

Duration: 45 minutes

Fabrikam would like to visualize the "hot" data showing the average temperature reported by each device over a 5-minute window in Power BI.

### Task 1: Create a Stream Analytics job for hot path processing to Power BI

1. In the [Azure Portal](https://portal.azure.com), select **+ Create a resource**, enter "stream analytics" into the Search the Marketplace box, select **Stream Analytics job** from the results, and select **Create**.

    ![In the Azure Portal, +Create a resource is highlighted, "stream analytics" is entered into the Search the Marketplace box, and Stream Analytics job is highlighted in the results.](media/create-resource-stream-analytics-job.png "Create Stream Analytics job")

2. On the New Stream Analytics Job blade, enter the following:

    * **Job name**: Enter hot-stream
    * **Subscription**: Select the subscription you are using for this hands-on lab
    * **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group
    * **Location**: Select the location you are using for resources in this hands-on lab
    * **Hosting environment**: Select Cloud

        ![The New Stream Analytics Job blade is displayed, with the previously mentioned settings entered into the appropriate fields.](media/stream-analytics-job-create.png "New Stream Analytics Job blade")

3. Select **Create**.

4. Once provisioned, navigate to your new Stream Analytics job in the portal.

5. On the Stream Analytics job blade, select **Inputs** from the left-hand menu, under Job Topology, then select **+Add stream input**, and select **IoT Hub** from the dropdown menu to add an input connected to your IoT Hub.

    ![Under Job Topology, Inputs is selected.](./media/image49.png "Job Topology section")

    ![On the Stream Analytics job blade, Inputs is selected under Job Topology in the left-hand menu, and +Add stream input is highlighted in the Inputs blade, and IoT Hub is highlighted in the drop down menu.](media/stream-analytics-job-inputs-add.png "Add Stream Analytics job inputs")

6. On the New Input blade, enter the following:

    * **Input alias**: Enter temps.
    * Choose **Select IoT Hub from your subscriptions**
    * **Subscription**: Select the subscription you are using for this hands-on lab
    * **IoT Hub**: Select the smartmeter-hub-SUFFIX IoT Hub
    * **Endpoint**: Select Messaging
    * **Shared access policy name**: Select service
    * **Consumer Group**: Leave set to $Default
    * **Event serialization format**: Select JSON
    * **Encoding**: Select UTF-8
    * **Event compression type**: Leave set to None

        ![IoT Hub New Input blade is displayed with the values specified above entered into the appropriate fields.](media/stream-analytics-job-inputs-add-iot-hub-input.png "IoT Hub New Input blade")

7. Select **Save**.

8. Next, select **Outputs** from the left-hand menu, under Job Topology, and select **+ Add**, then select **Power BI** from the drop down menu.

    ![Outputs is highlighted in the left-hand menu, under Job Topology, +Add is selected, and Power BI is highlighted in the drop down menu.](media/stream-analytics-job-outputs-add-power-bi.png "Add Power BI Output")

9. On the Power BI output blade, enter the following:

    * **Output alias**: Set to powerbi
    * Select **Authorize** to authorize the connection to your Power BI account. When prompted in the popup window, enter the account credentials you used to create your Power BI account in [Before the Hands-on Lab, Task 1](#task-1-provision-power-bi).

        ![Power BI new output blade. Output alias is selected and contains powerbi. Authorize button is highlighted.](media/stream-analytics-job-outputs-add-power-bi-authorize.png "Power BI new output blade")

    * For the remaining Power BI settings, enter the following:

        * **Group Workspace**: Select the default, My Workspace
        * **Dataset Name**: Enter avgtemps
        * **Table Name**: Enter avgtemps

10. Select **Save**.

    ![Power BI blade. Output alias is powerbi, dataset name is avgtemps, table name is avgtemps.](media/stream-analytics-job-outputs-add-power-bi-save.png "Add Power BI Output")

11. Next, select **Query** from the left-hand menu, under Job Topology.

    ![Under Job Topology, Query is selected.](./media/stream-analytics-job-query.png "Stream Analytics Query")

12. In the query text box, paste the following query.

    ``` sql
    SELECT AVG(temp) AS Average, id
    INTO powerbi
    FROM temps
    GROUP BY TumblingWindow(minute, 5), id
    ```

13. Select **Save**, and **Yes** when prompted with the confirmation.

    ![Save button on the Query blade is highlighted](./media/stream-analytics-job-query-save.png "Query Save button")

14. Return to the Overview blade on your Stream Analytics job and select **Start**.

    ![The Start button is highlighted on the Overview blade.](./media/stream-analytics-job-start.png "Overview blade start button")

15. In the Start job blade, select **Now** (the job will start processing messages from the current point in time onward).

    ![Now is selected on the Start job blade.](./media/stream-analytics-job-start-job.png "Start job blade")

16. Select **Start**.

17. Allow your Stream Analytics Job a few minutes to start.

18. Once the Stream Analytics Job has successfully started, verify that you are showing a non-zero amount of **Input Events** on the **Monitoring** chart on the overview blade. You may need to reconnect your devices on the Smart Meter Simulator and let it run for a while to see the events.

    ![The Stream Analytics job monitoring chart is diplayed with a non-zero amount of input events highlighted.](media/stream-analytics-job-monitoring-input-events.png "Monitoring chart for Stream Analytics job")

### Task 2: Visualize hot data with Power BI

1. Sign in to your Power BI subscription (<https://app.powerbi.com>) to see if data is being collected.

2. Select **My Workspace** on the left-hand menu, then select the **Datasets tab**, and locate the **avgtemps** dataset from the list.

    ![On the Power BI window, My Workspace is highlighted in the left pane, and the Datasets tab is highlighted in the right pane, and the avgtemps dataset is highlighted.](media/power-bi-workspaces-datasets-avgtemps.png "Power BI Datasets")

3. Select the Create Report button under the Actions column.

    ![On the Datasets tab, under Actions, the Create Report button is highlighted.](./media/power-bi-datasets-avgtemps-create-report.png "Datasets tab, Action column")

4. On the Visualizations palette, select **Stacked column chart** to create a chart visualization.

    ![On the Visualizations palette, the stacked column chart icon is highlighted.](./media/power-bi-visualizations-stacked-column-chart.png "Visualizations palette")

5. In the Fields listing, drag the **id** field, and drop it into the **Axis** field.

    ![Under Fields, an arrow points from the id field under avgtemps, to the same id field now located in the Visualizations listing, under Axis.](./media/power-bi-visualizations-stacked-column-chart-axis.png "Visualizations and Fields")

6. Next, drag the **average** field and drop it into the **Value** field.

    ![Under Fields, an arrow points from the average field under avgtemps, to the same id field now located in the Visualizations listing, under Value.](./media/power-bi-visualizations-stacked-column-chart-value.png "Visualizations and Fields")

7. Now, set the Value to **Max of average**, by click the down arrow next to **average**, and select **Maximum**.

    ![On the Value drop-down list, Maximum is highlighted.](./media/power-bi-visualizations-stacked-column-chart-value-maximum.png "Value drop-down list")

8. Repeat steps 5-8, this time adding a Stacked Column Chart for **Min of average**. (You may need to click on any area of white space on the report designer surface to deselect the Max of average by id chart visualization.)

    ![Min of average is added under Value.](./media/power-bi-visualizations-stacked-column-chart-value-minimum.png "Min of average")

9. Next, add a **table visualization**.

    ![On the Visualizations pallete, the table icon is highlighted.](./media/image69.png "Visualizations pallete")

10. Set the values to **id** and **Average of average**, by dragging and dropping both fields in the Values field, then selecting the dropdown next to average, and selecting Average.

    ![ID and Average of average now display under Values.](./media/power-bi-visualizations-table-average-of-average.png "Table Visualization values")

11. Save the report.

    ![Under File, Save is highlighted.](media/power-bi-save-report.png "Save report")

12. Enter the name "Average Temperatures," and select **Save**.

    ![The report name is set to Average Temperatures.](./media/power-bi-save-report-average-temperatures.png "Save your report")

13. Switch to **Reading View**.

    ![Power BI report Reading view button](media/power-bi-report-reading-view.png "Reading view button")

14. Within the report, select one of the columns to see the data for just that device.

    ![The report window has two bar graphs: Max of average by id, and Min of average by id. both bar charts list data for Device0, Device1, Device3, Device8, and Device9. Device1 is selected. On the right, a table displays data for Device1, with an Average of average value of 69.50.](./media/power-bi-report-reading-view-single-device.png "Report window")

## Exercise 4: Cold path data processing with Azure Databricks

Duration: 60 minutes

Fabrikam would like to be able to capture all the "cold" data into scalable storage so that they can summarize it periodically using a Spark SQL query.

### Task 1: Provision Event Hub

1. In the [Azure portal](https://portal.azure.com), select **+ Create a resource**, enter "event hub" into the Search the Marketplace box, select **Event Hubs** from the results, and select **Create**.

    ![In the Azure portal, +Create a resource is highlighted, "event hub" is entered into the Search the Marketplace box, and **Event Hubs** is highlighted in the results.](media/create-resource-event-hubs.png "Create Event Hubs")

2. On the Create Event Hubs Namespace blade, enter the following:

    * **Name**: Enter smartmetereventsSUFFIX
    * **Pricing tier**: Select Basic
    * **Subscription**: Select the subscription you are using for this hands-on lab
    * **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group
    * **Location**: Select the location you are using for resources in this hands-on lab
    * **Throughput Units**: Leave set to 1

        ![On the Create Event Hubs Namespace blade, the values specified above are entered into the appropriate fields.](media/event-hubs-namespace-create.png "Create Event Hubs Namespace")

3. Select **Create**.

4. Once provisioning completes, navigate to the event hub in the Azure portal.

5. On the Overview blade, select **+ Event Hub**.

    ![On the Overview blade of the Event Hubs Namespace, +Event Hubs is highlighted.](media/event-hubs-namespace-add-event-hub.png "Add Event Hub")

6. On the Create Event Hub blade, enter **smart-meter-events** for the Name, then select **Create**.

    !["On the Create Event Hub blade, "smart-meter-events" is entered into the name field, and partition count is set to 2.](media/event-hubs-namespace-create-event-hub.png "Create Event Hub")

### Task 2: Create a messaging endpoint for cold path processing to Power BI

In this task, you will create an endpoint and route for cold data processing.

1. In the [Azure portal](https://portal.azure.com), navigate to your IoT Hub, select **Endpoints** under Messaging in the left-hand menu, and select **+Add**.

    ![In the Azure portal, the IoT Hub blade is displayed. Endpoints is highlighted under Messaging in the left-hand menu, and +Add is highlighted on the Endpoints blade.](media/iot-hub-messaging-endpoints.png "IoT Hub Messaging Endpoints")

2. On the Add endpoint blade, enter the following:

    * **Name**: Enter cold-path-endpoint
    * **Endpoint type**: Select Azure Storage Container
    * Select **Azure Storage Container**, select **+ Storage account**, then enter **smartmetersSUFFIX** for the name, and select **OK**.

        ![The Add endpoint blade for IoT messaging is displayed, with the values specified above entered into the appropriate fields.](media/iot-hub-messaging-endpoints-add-cold-path.png "IoT Hub Messaging Add endpoint")

    * Next, select the **smartmetersSUFFIX** account the list of storage accounts, then select **+ Container**, and enter **smartmeters** into the name field, and select **OK**.

        ![The Storage account blade is selected, and the smartmetersSUFFIX account is highlighted. On the Containers blade, +Container is highlighted, and smartmeters is entered into the Name field.](media/iot-hub-messaging-endpoints-add-cold-path-storage-container.png "IoT Hub Storage accounts Containers")

    * Select the **smartmeters** container and select **Select**.
    * Back on the Add endpoint blade, set the **Blob file name format** to **{iothub}-{partition}/{YYYY}-{MM}-{DD}-{HH}-{mm}**.

        ![On the Add endpoint blade, the values specified above are entered into the appropriate fields.](media/iot-hub-messaging-endpoint-add-cold-path-final.png "Add endpoint")

3. Select **OK**.

4. Now, select **Routes** from the left-hand menu of the IoT Hub blade, then select **+Add**.

    ![In the Azure portal, the IoT Hub blade is displayed. Routes is highlighted under Messaging in the left-hand menu, and +Add is highlighted on the Routes blade.](media/iot-hub-messaging-routes.png "IoT Hub Messaging Routes")

5. On the Create a new route blade, enter the following:

    * **Name**: enter cold-path-route
    * **Data source**: Select Device Messages
    * **Endpoint**: Select cold-data-endpoint
    * **Query string**: Leave blank (this will match all messages)

        ![The Create a new route blade is displayed, and the values specified above are entered into the appropriate fields.](media/iot-hub-messaging-routes-add.png "Add new messaging route")

6. Select **Save**.

### TODO: DELETE: Task 1: Create the Stream Analytics job for cold path processing

To capture all metrics for the cold path, set up another Stream Analytics job that will write all events to Blob storage for analyses by Spark running on HDInsight.

1.  In your browser, navigate to the **Azure Portal** (<https://portal.azure.com>).

2.  Select **+New**, **Data + Analytics**, then select **Stream Analytics job**.
    
    ![In the Azure Portal, in the left pane, New is selected. In the New pane, Data + Analytics and Stream Analytics job are highlighted.](./media/image47.png "Azure Portal")

3.  On the New Stream Analytics Job blade, enter the following:

  * Job Name: Enter **cold-stream**.

  * Subscription: Choose the same subscription you have been using thus far.

  * Resource Group: Choose the **iot-hol** Resource Group.

  * Location: Choose the same Location as you have for your other resources.

  * Hosting environment: Select **Cloud**.

        ![The New Stream Analytics Job blade fields display with the previously mentioned settings.](./media/image75.png "New Stream Analytics Job blade")

  * Select **Create**.

4.  Once provisioned, navigate to your new Stream Analytics job in the portal.

5.  Select **Inputs** on the left-hand menu, under Job Topology.

    ![Under Job Topology, Inputs is selected.](./media/image49.png "Job Topology Section")

6.  On the Inputs blade, select **+Add**, then **IoT Hub** to add an input connected to your IoT Hub.
    
    ![Select + Add stream input, then select IoT Hub from the dropdown menu](./media/image50.png "Add stream input")

7.  On the New Input blade, enter the following:

  * Input Alias: Set the value to **iothub**.

  * Choose **Select IoT hub from your subscriptions**.

  * IoT Hub: Select your existing IoT Hub, **smartmeter-hub**.

  * Endpoint: Choose **Messaging**.

  * Shared Access Policy Name: Set to **Service**.

  * Consumer Group: Leave as **\$Default**.

  * Event serialization format: Choose **JSON**.

  * Encoding: Choose **UTF-8**.

  * Event compression type: Leave set to **None**.

        ![Form for adding a new IoT Hub input to Stream Analytics](./media/image76.png "IOT hub new input blade")

  * Select **Create**.

8.  Now, select **Outputs** from the left-hand menu, under Job Topology.

    ![Under Job Topology, Outputs is selected.](./media/image52.png "Job Topology Section")

9.  In the Outputs blade, select **+Add**, then **Blob storage** to add the output destination for the query.
    
    ![Outputs blade add blob storage](./media/image77.png "Outputs blade add blob storage")

10. On the New output blade, enter the following:

  * Output alias: Set to **blobs**.

  * Choose **Select blob storage from your subscriptions**.

  * Storage account: Choose the storage account name you used for HDInsight in Before the hands-on lab, Task 2, Step 5c.

  * Container: Set to **iotcontainer**.

  * Path pattern: Enter **smartmeters/{date}/{time}**.

  * Date format: Select **YYYY/MM/DD**.

  * Time format: Select **HH**.

  * Event serialization formation: Select **CSV**.

  * Delimiter: Select **comma (,)**.

  * Encoding: Select **UTF-8**.

        ![Blob storage blade with storage account, path pattern and event serialzation format complted.](./media/image78.png "Blob Storage blade")

  * Select **Save**.

11. Next, select **Query** from the left-hand menu, under Job Topology.
    
    ![Under Job Topology, Query is selected.](./media/image56.png "Job Topology Section")

12. In the query text box, paste the following query.
    ```
    SELECT
          *
    INTO 
          blobs
    FROM 
          iothub
    ```

13. Select **Save,** and **Yes** when prompted with the confirmation.

    ![Save button](./media/image57.png "Save button")

14. Return to the Overview blade on your Stream Analytics job, and select **Start**.

    ![Start button](./media/image58.png "Start button")

15. In the Start job blade, select **Now** (the job will start processing messages from the current point in time onward).
    
    ![Now button](./media/image59.png "Now button")

16. Select **Start**.

17. Allow your Stream Analytics Job a few minutes to start.

18. Once the Stream Analytics Job has successfully started, verify that you are showing a non-zero amount of **Input Events** on the **Monitoring** chart on the overview blade. You may need to reconnect your devices on the Smart Meter Simulator and let it run for a while to see the events.

    ![The Monitoring chart now lists 88 Input Events, 88 Output Events, and 0 runtime errors.](./media/image79.png "Monitoring chart")

### Task 3: Verify CSV files in blob storage

In this task, we are going to verify that the CSV file is being written to blob storage. (Note, this can be done via Visual Studio, or using the Azure portal. For this lab, we will perform the task using Visual Studio.)

1. Within Visual Studio on your Lab VM, select the **View** menu, then select **Cloud Explorer**.

    ![On the Visual Studio View menu, Cloud Explorer is highlighted.](./media/visual-studio-menu-view-cloud-explorer.png "Visual Studio View menu")

2. In **Cloud Explorer**, select Account Management, and connect to your Microsoft Azure Subscription.

    ![The Cloud Explorer window displays, and the Account management icon is highlighted.](./media/visual-studio-cloud-explorer-account-management.png "Cloud Explorer Account Management")

3. If prompted, sign into your Azure account.

4. Allow Cloud Explorer about 30 seconds to load your subscription resources.

5. Expand your Azure account, then expand **Storage Accounts**, expand the smartmetersSUFFIX storage account, then right-click the smartmeters container, and select **Open**. It may take a few moments to load your storage accounts.

    ![Storage accounts is expanded in the Visual Studio Cloud Explorer, with the smartmetersSUFFIX account is expanded, and the Open menu item highlighted for the smartmeters container.](media/visual-studio-cloud-explorer-storage-accounts.png "Cloud Explorer Storage Accounts")

6. Verify files are being written to Blob storage and take note of the path to one of the files (the files should be located underneath the smartmeters container).

    ![Files are listed in the blob storage account, as written by the cold path route in IoT Hub Messaging.](media/smart-meters-cold-path-files.png "Smart meters files in blob storage")

### Task 3: Update pandas version on the Spark cluster

In this task you will connect SSH into your HDInsight cluster, and update the version of pandas that Jupyter Notebook uses. This task is to address errors that are displayed because the autovizwidget in Jupyter needs a later version of pandas that has the API module.

1.  In the Azure portal, navigate to the blade for your Spark Cluster, under your HDInsight Cluster.

2.  On the cluster's overview blade, select **Secure Shell (SSH)** from the toolbar.
    
    ![On the Overview blade toolbar, Secure Shell (SSH) is highlighted.](./media/image85.png "Overview blade toolbar")

3.  On the SSH + Cluster login blade, select your cluster from the Hostname drop down, then select the **copy button** next to the text box.
    
    ![On the SSH + Cluster login blade, under Hostname, the cluster name is highlighted, as is the copy button next to the textbox.](./media/image86.png "SSH + Cluster login blade")

4.  On your Lab VM, select your open Git Bash client (or open a new one if you closed it).

1.  At the command prompt, paste the SSH connection string you copied in step 3, above. When prompted about continuing, type "yes."
    
    ![The Git Bash Window displays the SSh connection string, and the prompt, \"Are you sure you want ot continue connecting?\" is highlighted.](./media/image87.png "Git Bash Window")

6.  Enter your password, Password.1!!, when prompted, and press **Enter**.
    
    ![The Git Bash Window now has the prompt for the password highlighted.](./media/image88.png "Git Bash Window")

7.  Once logged in, you will be presented with a command prompt, similar to the following:
    
    ![The Git Bash Window now displays with the following prompt: sshuser\@hn0-iothan:\~\$](./media/image89.png "Git Bash Window")

8.  At the command prompt, enter the following command to install the latest version of pandas, which has the API module required by the autovizwidget.
    ```
    sudo -HE /usr/bin/anaconda/bin/conda install pandas
    ```
9.  When prompted to Proceed, type "y."
    
    ![In the Git Bash Window, the following line is highlighted: Proceed (\[y\]n)? y](./media/image90.png "Git Bash Window")

### Task 4: Process with Spark SQL 

1.  Navigate to the blade for your Spark Cluster in the Azure Portal, under you HDInsight Cluster.

2.  Under Quick Links, click **Cluster Dashboard**.
    
    ![Under Quick links, the Cluster dashboard option is highlighted.](./media/image91.png "Quick links")

3.  On the Cluster Dashboards blade, select **Jupyter Notebook**. If prompted, log in with admin credentials you provided when creating the cluster (username: **admin**, password: **Password.1!!**).
    
    ![On the Cluster Dashboards blade, the Jupyter Notebook option is highlighted.](./media/image92.png "Cluster Dashboards blade")

4.  From the navigation bar in the Jupyter site, select **New** and then **Spark.**
    
    ![On the Jupyter site navigation bar, New is selected, and on its drop-down menu, Spark is selected. A pop-up message says, \"Create a new notebook with Spark."](./media/image93.png "Jupyter site navigation bar")

5.  In the first text area (referred to as a paragraph in notebook jargon), enter the following **Scala code** that will load, parse, and save your batch scored telemetry data as a table that you can later query using Spark SQL. Before executing, make sure to replace the highlighted text with the correct path to your telemetry file, noted previously in the Cloud Explorer in Visual Studio.
    ```
    import spark.implicits._

    val rawText = spark.read.text("wasb:///smartmeters/2017/08/20/19/1008078303_90a84f7aba614d1fa4688cbda1de3846_1.csv")
    case class SmartMeterMetrics(id:String,time:String,temp:Integer)
    val telemetryRDD = rawText.map(row => row.getString(0).split(",")).filter(s=>s(0) != "id").map(
        s => SmartMeterMetrics(s(0), s(1), s(2).toInt)
    )
    val telemetryDF = telemetryRDD.toDF()
    telemetryDF.write.saveAsTable("SmartMeters")
    ```

1.  Next, click the **Run** icon in the toolbar to execute this code and create the **SmartMeters** table
    
    . ![The Run icon is selected on the toolbar.](./media/image94.png "Run icon")

7.  The block is finished running when the In\[\*\] changes to In\[1\]\
    
    ![In the top half of this screen capture, code displays. In the bottom half of the screen capture, a SmartMeters table displays.](./media/image95.png "SmartMeters table")

    You may see the below message. You can proceed.
    
    ![The message states that the SmartMeters table already exists.](./media/image96.png "Message")

8.  In the second cell, enter the following SQL query and run it.
    ```
    %%sql
    select id, count(*) as count, avg(temp) averageTemp from SmartMeters group by id order by id
    ```

    You will see a table like the following:

    ![A table with three columns displays: id, count, and average temp. Devices 0 through 6 have counts varying between 59 and 60, and average Temps between 69.0 and 69.9.](./media/image97.png "Table")

9.  Next, create a table that will summarize the telemetry collected using the previous query. In a new paragraph, try running the following query:
    ```
    //query the table to create a summary result set
    val summary = spark.sql("select id, count(*) as count, avg(temp) averageTemp from SmartMeters group by id order by id")

    //save the new pre-computed view
    summary.write.saveAsTable("DeviceSummary")
    ```

10. Next, query from this summary table by executing the following query.
    ```
    %%sql
    select * from DeviceSummary
    ```

11. In the results, click the **Bar** button.

12. In the X dropdown, select **id**.

13. In the Y dropdown, select **averageTemp**.

14. In the Func dropdown, select **Avg**.

15. Check the box for **Log scale** **Y.**

16. Observe the results graphed as a column chart, where each column represents a device's average temperature.
    
    ![A bar chart displays, with columns displaying for devices 0, 1, 3, 8, and 9. The chart indicates that Devices 0 and 1 have lower average temperatures, and that Device3 has the highest average temperature. Above the chart, for Type, Bar is highlighted. Under Encoding, for X, id is selected. For Y, averageTemp is selected. for Func, Avg is selected. The checkbox for Log scale Y is selected. ](./media/image98.png "Column chart")

## Exercise 6: Reporting device outages with IoT Hub Operations Monitoring

Duration: 20 minutes

Fabrikam would like to be alerted when devices disconnect and fail to reconnect after a period. Since they are already using PowerBI to visualize hot data, they would like to see a list of any of these devices in a report.

### Task 1: Enable verbose connection monitoring on the IoT Hub

To keep track of device connects and disconnects, we first need to enable verbose connection monitoring.

1.  In your browser, navigate to the **Azure Portal** (<https://portal.azure.com)>.

2.  Open the IoT Hub you provisioned earlier, **smart-meter**.

3.  Under SETTINGS in the left-hand menu, click on **Operations monitoring**.
    
    ![Under Settings, Operations monitoring is selected.](./media/image99.png "Settings section")

4.  Select **Verbose** for **Log events when a device connects or disconnects from the IoT Hub**.

    ![In the Monitoring categories section, Verbose is selected under Log events when a device connects or disconnects from the IoT Hub.](./media/image100.png "Monitoring categories section")

5.  Click **Save**.

### Task 2: Collect device connection telemetry with the hot path Stream Analytics job

Now that the device connections are being logged, update your hot path Stream Analytics job (the first one you created) with a new input that ingests device telemetry from Operations Monitoring. Next, create a query that joins all connected and disconnected events with a DATEDIFF function that only returns devices with a disconnect event, but no reconnect event within 120 seconds. Output the events to Power BI.

1.  In your browser, navigate to the **Azure Portal** (<https://portal.azure.com)>.

2.  Open the **hot-stream** Stream Analytics job (the first one you created).

3.  Stop the job if it is currently running, from the Overview blade, by selecting **Stop**, then **Yes** when prompted.
    
    ![The Stop button is highlighted on the Overview blade top menu.](./media/image101.png "Overview blade Stop button")

4.  Select **Inputs** on the left-hand menu, under Job Topology.

    ![Under Job Topology, Inputs is selected.](./media/image49.png "Job Topology Section")

5.  On the Inputs blade, select **+Add**, then **IoT Hub** to add an input connected to your IoT Hub.
    
    ![Add new IoT Hub stream input to Stream Analytics](./media/image50.png "Job topolog add stream input hub")

6.  On the New Input blade, enter the following:

  * Input Alias: Set the value to **connections**.

  * Import Option: Choose **Select IoT hub from your subscriptions**.

  * IoT Hub: Select your existing IoT Hub, **smartmeter-hub**.

  * Endpoint: Choose **Operations monitoring**.

  * Shared Access Policy Name: Set to **Service**.

  * Consumer Group: Leave as **\$Default**.

  * Event serialization format: Choose **JSON**.

  * Encoding: Choose **UTF-8**.

  * Event compression type: Leave set to **None**.

        ![Create new IoT Hub input form for Stream Analytics](./media/image102.png "IoT hub new input blade")

  * Select **Save**.

7. Now, select **Outputs** from the left-hand menu, under Job Topology.

    ![Under Job Topology, Outputs is selected.](./media/image52.png "Job Topology Section")

8. In the Outputs blade, select **+Add**, the **Power BI**, to add the output destination for the query.
    
    ![Select +Add, then Power BI within the outputs blade of your Stream Analytics instance](./media/image53.png "Outputs blade add Power BI")

9. On the New output blade, enter the following:

  * Set the **Output alias** to **powerbi-outage**.

        ![Enter powerbi-outage for the output alias, then select Authorize to authenticate to your Power BI subscription](./media/image103.png "Poewr BI new output")

  * Select **Authorize** under Authorize Connection.

    Follow the on-screen prompts to log on to your Power BI account.

    After authenticating, complete the remaining fields as follows:

  * Group Workspace: Leave set to **My Workspace**.

  * Dataset Name: Enter **deviceoutage**

  * Table Name: Enter **deviceoutage**

  * Select **Save**.

        ![Complete the form by entering deviceoutage as both the Dataset name and Table name. Select the My workspace Group workspace option](./media/image104.png "Power BI new input ")

10. Next, select **Query** from the left-hand menu, under Job Topology.
    
    ![Under Job Topology, Query is selected.](./media/image56.png "Job Topology Section")

11. We will replace the hot path query, which selects the averages of the temperatures into the PowerBI output, with queries that perform the following:

  * Select **device disconnection events**.

  * Select **device connection events**.

  * Join these two streams together using the Stream Analytics DATEDIFF operation on the LEFT JOIN, and then filter out any records where there was a match. This gives us devices that had a disconnect event, but no corresponding connect event within 120 seconds. Output to the Service Bus.

  * Execute the original hot path query.

12. Replace the existing query with the following, and click **Save** in the **command bar** at the top. (Be sure to substitute in your output aliases and input aliases):
    ```
    WITH
    Disconnected AS (
    SELECT *
    FROM connections TIMESTAMP BY [Time]
    WHERE OperationName = 'deviceDisconnect'
        AND Category = 'Connections'
    ),
    Connected AS (
    SELECT *
    FROM connections TIMESTAMP BY [Time]
    WHERE OperationName = 'deviceConnect'
        AND Category = 'Connections'
    )

    SELECT Disconnected.DeviceId, Disconnected.Time
    INTO [powerbi-outage] 
    FROM Disconnected
    LEFT JOIN Connected 
        ON DATEDIFF(second, Disconnected, Connected) BETWEEN 0 AND 120
        AND Connected.deviceId = Disconnected.deviceId
    WHERE Connected.DeviceId IS NULL;

    SELECT AVG(temp) AS Average, id
    INTO powerbi
    FROM temps
    GROUP BY TumblingWindow(minute, 5), id;
    ```

13. Select **Save,** and **Yes** when prompted with the confirmation.

    ![Save button](./media/image57.png "Save button")

14. Return to the Overview blade on your Stream Analytics job, and select **Start**.

    ![The start button on the Overview blade is highlighted](./media/image58.png "Overview blade start button")

15. In the Start job blade, select **Now** (the job will start processing messages from the current point in time onward).
    
    ![Next to Job output start time, the Now button is selected.](./media/image59.png "Now button")

16. Select **Start**.

17. Allow your Stream Analytics Job a few minutes to start.

### Task 3: Test the device outage notifications

Register and activate a few devices on the Smart Meter Simulator, then connect them. Deactivate them without reconnecting in order for them to show up in the device outage report we will create in the next task.

1.  Run the Smart Meter Simulator from Visual Studio.

2.  Click the **Register** button.
    
    ![Register button](./media/image105.png "Register button")

3.  Click on 3 of the windows to highlight them.
    
    ![In the SmartMeter Simulator, three white windows display.](./media/image106.png "SmartMeter Simulator")

4.  Click the **Activate** button.
    
    ![Activate button](./media/image107.png "Activate button")

5.  Click the **Connect** button.
    
    ![Connect button](./media/image108.png "Connect button")

6.  After a few seconds, click **Disconnect**.
    
    ![Disconnect button](./media/image109.png "Disconnect button")

7.  Click **Unregister**.
    
    ![Unregister button](./media/image110.png "Unregister button")

### Task 4: Visualize disconnected devices with Power BI

1.  Log on to **Power BI** to see if data is being collected.

2.  As done previously, select My Workspace on the left-hand menu, then select the Datasets tab. A new dataset should appear, named **deviceoutage**. (It is starred to indicate it is new) If you do not see the dataset, you may need to connect your devices on the Smart Meter Simulator, then disconnect and unregister them and wait up to 5 minutes.
    
    ![On the Power BI window, My Workspace is highlighted in the left pane, and the Datasets tab is highlighted in the right pane. Under Name, deviceoutage is highlighted.](./media/image111.png "Power BI window")

3.  Select the **Create report** icon under actions for the dataset.
    
    ![Next to deviceoutage, the Create report icon is highlighted.](./media/image112.png "Create report icon")

4.  Add a **Table visualization**.

    ![The table icon is highlighted on the Visualizations palette.](./media/image113.png "Visualizations palette")

5.  Select the **deviceid** and **time** fields, which will automatically be added to the table. You should see the Device Id of each of the devices you connected, and then disconnected for more than 2 minutes.
    
    ![In the Fields listing, under deviceoutage, deviceid and time are both selected.](./media/image114.png "Fields listing")\
    
    ![DeviceId and Time results display for Device one, Device five, and Device 8.](./media/image115.png "DeviceId and Time results")

6.  Save the report as **Disconnected Devices**.

    ![Screenshot of the Save this report option.](./media/image116.png "Save this report option")

7.  Switch to **Reading View**.

    ![Reading View button](./media/image73.png "Reading View button")

8.  Within the report, click the column headers to sort by device or date. You may run a few more tests with the Smart Meter Simulator and periodically refresh the report to see new devices.

    ![This report has two columns: DeviceID, and Time. Data is sorted by time.](./media/image117.png "Report")

## After the hands-on lab 

Duration: 10 minutes

In this exercise, attendees will deprovision any Azure resources that were created in support of the lab.

### Task 1: Delete the resource group

1.  Using the Azure portal, navigate to the Resource group you used throughout this hands-on lab by selecting **Resource groups** in the left menu.

2.  Search for the name of your research group, and select it from the list.

3.  Select **Delete** in the command bar, and confirm the deletion by re-typing the Resource group name, and selecting **Delete**.

