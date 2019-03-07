![](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png 'Microsoft Cloud Workshops')

<div class="MCWHeader1">
Internet of Things
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>

<div class="MCWHeader3">
November 2018
</div>

Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/en-us/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

- [Internet of Things hands-on lab step-by-step](#internet-of-things-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Overview](#overview)
  - [Solution architecture](#solution-architecture)
  - [Requirements](#requirements)
  - [Exercise 1: IoT Hub provisioning](#exercise-1-iot-hub-provisioning)
    - [Task 1: Provision IoT Hub](#task-1-provision-iot-hub)
    - [Task 2: Configure the Smart Meter Simulator](#task-2-configure-the-smart-meter-simulator)
  - [Exercise 2: Completing the Smart Meter Simulator](#exercise-2-completing-the-smart-meter-simulator)
    - [Task 1: Implement device management with the IoT Hub](#task-1-implement-device-management-with-the-iot-hub)
    - [Task 2: Implement the communication of telemetry with IoT Hub](#task-2-implement-the-communication-of-telemetry-with-iot-hub)
    - [Task 3: Verify device registration and telemetry](#task-3-verify-device-registration-and-telemetry)
  - [Exercise 3: Hot path data processing with Stream Analytics](#exercise-3-hot-path-data-processing-with-stream-analytics)
    - [Task 1: Create a Stream Analytics job for hot path processing to Power BI](#task-1-create-a-stream-analytics-job-for-hot-path-processing-to-power-bi)
    - [Task 2: Visualize hot data with Power BI](#task-2-visualize-hot-data-with-power-bi)
  - [Exercise 4: Cold path data processing with Azure Databricks](#exercise-4-cold-path-data-processing-with-azure-databricks)
    - [Task 1: Create a storage account](task-1-create-a-storage-account)
    - [Task 2: Create the Stream Analytics job for cold path processing](#task-2-create-the-stream-analytics-job-for-cold-path-processing)
    - [Task 3: Verify CSV files in blob storage](#task-3-verify-csv-files-in-blob-storage)
    - [Task 4: Process with Spark SQL](#task-4-process-with-spark-sql)
  - [Exercise 5: Reporting device outages with IoT Hub Operations Monitoring](#exercise-5-reporting-device-outages-with-iot-hub-operations-monitoring)
    - [Task 1: Enable verbose connection monitoring on the IoT Hub](#task-1-enable-verbose-connection-monitoring-on-the-iot-hub)
    - [Task 2: Collect device connection telemetry with the hot path Stream Analytics job](#task-2-collect-device-connection-telemetry-with-the-hot-path-stream-analytics-job)
    - [Task 3: Test the device outage notifications](#task-3-test-the-device-outage-notifications)
    - [Task 4: Visualize disconnected devices with Power BI](#task-4-visualize-disconnected-devices-with-power-bi)
  - [After the hands-on lab](#after-the-hands-on-lab)
    - [Task 1: Delete the resource group](#task-1-delete-the-resource-group)

# Internet of Things hands-on lab step-by-step

If you have not yet completed the steps to set up your environment in [Before the hands-on lab setup guide](./Before%20the%20HOL%20-%20Internet%20of%20Things.md), you will need to do that before proceeding.

## Abstract and learning objectives

In this hands-on lab, you will construct an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path.

At the end of this hands-on lab, you will be better able to build an IoT solution implementing device registration with the IoT Hub Device Provisioning Service and visualizing hot data with Power BI.

## Overview

Fabrikam provides services and smart meters for enterprise energy (electrical power) management. Their "_You-Left-The-Light-On_" service enables the enterprise to understand their energy consumption.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

![Diagram of the preferred solution. From a high-level, the commerce solution uses an API App to host the Payments web service with which the Vending Machine interacts to conduct purchase transactions. The Payment Web API invokes a 3rd party payment gateway as needed for authorizing and capturing credit card payments, and logs the purchase transaction to SQL DB. The data for these purchase transactions is stored using an In-Memory table with a Columnar Index, which will support the write-heavy workload while still allowing analytics to operate, such as queries coming from Power BI Desktop.](./media/preferred-solution-architecture.png 'Preferred high-level architecture')

Messages are ingested from the Smart Meters via IoT Hub and temporarily stored there. A Stream Analytics job pulls telemetry messages from IoT Hub and sends the messages to two different destinations. There are two Stream Analytics jobs, one that retrieves all messages and sends them to Blob Storage (the cold path), and another that selects out only the important events needed for reporting in real time (the hot path). Data entering the hot path will be reported on using Power BI visualizations and reports. For the cold path, Azure Databricks can be used to apply the batch computation needed for the reports at scale.

Other alternatives for processing of the ingested telemetry would be to use an HDInsight Storm cluster, a WebJob running the EventProcessorHost in place of Stream Analytics, or HDInsight running with Spark streaming. Depending on the type of message filtering being conducted for hot and cold stream separation, IoT Hub Message Routing might also be used, but this has the limitation that messages follow a single path, so with the current implementation, it would not be possible to send all messages to the cold path, while simultaneously sending some of the same messages into the hot path. An important limitation to keep in mind for Stream Analytics is that it is very restrictive on the format of the input data it can process: the payload must be UTF8 encoded JSON, UTF8 encoded CSV (fields delimited by commas, spaces, tabs, or vertical pipes), or AVRO, and it must be well formed. If any devices transmitting telemetry cannot generate output in these formats (e.g., because they are legacy devices), or their output can be not well formed at times, then alternatives that can better deal with these situations should be investigated. Additionally, any custom code or logic cannot be embedded with Stream Analytics---if greater extensibility is required, the alternatives should be considered.

> **Note**: The preferred solution is only one of many possible, viable approaches.

## Requirements

- Microsoft Azure subscription must be pay-as-you-go or MSDN.
  - Trial subscriptions will not work.
- A virtual machine configured with:
  - Visual Studio Community 2017 15.6 or later
  - Azure SDK 2.9 or later (Included with Visual Studio 2017)
- A running Azure Databricks cluster (see [Before the hands-on lab](./Before%20the%20HOL%20-%20Internet%20of%20Things.md))

## Exercise 1: IoT Hub provisioning

Duration: 15 minutes

In your architecture design session with Fabrikam, it was agreed that you would use an Azure IoT Hub to manage both the device registration and telemetry ingest from the Smart Meter Simulator. Your team also identified the Microsoft provided Device Explorer project that Fabrikam can use to view the list and status of devices in the IoT Hub registry.

### Task 1: Provision IoT Hub

In these steps, you will provision an instance of IoT Hub.

1. In your browser, navigate to the [Azure portal](https://portal.azure.com), select **+Create a resource** in the navigation pane, enter "iot" into the Search the Marketplace box, select **IoT Hub** from the results, and select **Create**.

   ![+Create a resource is highlighted in the navigation page of the Azure portal, and "iot" is entered into the Search the Marketplace box. IoT Hub is highlighted in the search results.](./media/create-resource-iot-hub.png 'Create an IoT Hub')

2. On the IoT Hub blade Basics tab, enter the following:

   - **Subscription**: Select the subscription you are using for this hands-on lab.

   - **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group.

   - **Region**: Select the location you are using for this hands-on lab.

   - **IoT Hub Name**: Enter a unique name, such as smartmeter-hub-SUFFIX.

     ![The Basics blade for IoT Hub is displayed, with the values specified above entered into the appropriate fields.](./media/iot-hub-basics-blade.png 'Create IoT Hub Basic blade')

   - Select **Next: Size and Scale**.

   - On the Size and scale blade, accept the default Pricing and scale tier of S1: Standard tier, and select **Review + create**.

   - Select **Create** on the Review + create blade.

3. When the IoT Hub deployment is completed, you will receive a notification in the Azure portal. Select **Go to resource** in the notification.

   ![Screenshot of the Deployment succeeded message, with the Go to resource button highlighted.](./media/iot-hub-deployment-succeeded.png 'Deployment succeeded message')

4. From the IoT Hub's Overview blade, select **Shared access policies** under Settings on the left-hand menu.

   ![Screenshot of the Overview blade, settings section. Under Settings, Shared access policies is highlighted.](./media/iot-hub-shared-access-policies.png 'Overview blade, settings section')

5. Select **iothubowner** policy.

   ![The Azure portal is shown with the iothubowner selected.](./media/iot-hub-shared-access-policies-iothubowner.png 'IoT Hub Owner shared access policy')

6. In the **iothubowner** blade, select the Copy button to the right of the **Connection string - primary key** field. You will be pasting the connection string value into a TextBox's Text property value in the next task.

   ![Screenshot of the iothubowner blade. The connection string - primary key field is highlighted.](./media/iot-hub-shared-access-policies-iothubowner-blade.png 'iothubowner blade')

### Task 2: Configure the Smart Meter Simulator
> **Note**: If you have problem to create the Lab VM with Visual Studio, please download the binary vesion [SmartMeterSimulator](https://tinyurl.com/kriot20190306) and you may skip this task and `Exercise 2`. However, you should learn how to use `SmartMeterSimulator` following the `Task 3: Verify device registration and telemetry` under `Exercise 2`


If you want to save this connection string with your project (in case you stop debugging or otherwise close the simulator), you can set this as the default text for the text box. Follow these steps to configure the connection string:

1. Return to the `SmartMeterSimulator` solution in Visual Studio on your Lab VM.

2. In the Solution Explorer, double-click `MainForm.cs` to open it. (If the Solution Explorer is not in the upper left corner of your Visual Studio instance, you can find it under the View menu in Visual Studio.)

   ![In the Visual Studio Solution Explorer window, SmartMeterSimulator is expanded, and under it, MainForm.cs is highlighted.](media/visual-studio-solution-explorer-mainform-cs.png 'Visual Studio Solution Explorer')

3. In the Windows Forms designer surface, click the **IoT Hub Connection String TextBox** to select it.

   ![The Windows Form designer surface is opened to the MainForm.cs tab. The IoT Hub Connection String is highlighted, but is empty.](./media/smart-meter-simulator-iot-hub-connection-string.png 'Windows Form designer surface')

4. In the Properties panel, scroll until you see the **Text** property. Paste your IoT Hub connection string value copied in step 6 of the previous task into the value for the Text property. (If the properties window is not visible below the Solution Explorer, right-click the TextBox, and select **Properties**.)

   ![In the Properties panel, the Text property is highlighted, and is set to HostName=smartmeter-hub.](./media/smart-meter-simulator-iot-hub-connection-string-text-property.png 'Solution Explorer')

5. Your connection string should now be present every time you run the Smart Meter Simulator.

   ![The Windows Form designer surface is opened to the MainForm.cs tab. The IoT Hub Connection String now displays.](./media/smart-meter-simulator-iot-hub-connection-string-populated.png 'IoT Hub Connection String dialog')

6. Save `MainForm.cs`.

## Exercise 2: Completing the Smart Meter Simulator

Duration: 60 minutes

Fabrikam has left you a partially completed sample in the form of the Smart Meter Simulator solution. You will need to complete the missing lines of code that deal with device registration management and device telemetry transmission that communicate with your IoT Hub.

### Task 1: Implement device management with the IoT Hub

1. In Visual Studio on your Lab VM, use Solution Explorer to open the file `DeviceManager.cs`.

2. From the Visual Studio **View** menu, click **Task List**.

   ![On the Visual Studio View menu, Task List is selected.](media/visual-studio-view-menu-task-list.png 'Visual Studio View menu')

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

### Task 2: Implement the communication of telemetry with IoT Hub

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

   > **Note**: If you receive the following error after building the solution, complete the steps that follow: `Couldn't process file MainForm.resx due to its being in the Internet or Restricted zone or having the mark of the web on the file. Remove the mark of the web if you want to process these files.`

   - Open Windows Explorer and navigate to the starter project folder: `C:\SmartMeter\Hands-on lab\lab-files\starter-project\SmartMeterSimulator\`.
   - Right-click on the `MainForm.resx` file, then select Properties.
   - Check the **Unblock** checkbox on the bottom of the General tab, then click Apply then OK.

   ![Right-click MainForm.resx, go to Properties, then check the box next to Unblock](media/unblock-file.png 'Unblock file')

   - Close and reopen Visual Studio. You should now be able to successfully build.

2. Run the Smart Meter Simulator, by selecting the green Start button on the Visual Studio toolbar.

   ![The green Start button is highlighted on the Visual Studio toolbar.](media/visual-studio-toolbar-start.png 'Visual Studio toolbar')

3. Select **Register** on the Smart Meter Simulator dialog, which should cause the windows within the building to change from black to gray.

   ![In addition to the IoT Hub Connection String, the Smart Meter Simulator has two buildings with 10 windows. The color of the windows indicating the status of the devices. Currently, all windows are gray.](media/smart-meter-simulator-register.png 'Fabrikam Smart Meter Simulator')

4. Select a few of the windows. Each represents a device for which you want to simulate device installation. The selected windows should turn yellow.

   ![The Smart Meter Simulator now has three white windows, with the other seven remaining gray.](media/smart-meter-simulator-window-select.png 'Fabrikam Smart Meter Simulator')

5. Select **Activate** to simulate changing the device status from disabled to enabled in the IoT Hub Registry. The selected windows should turn green.

   ![On the Smart Meter Simulator, the Activate button is highlighted, and the three white windows have now turned to green.](media/smart-meter-simulator-activate.png 'Fabrikam Smart Meter Simulator')

6. At this point, you have registered 10 devices (the gray windows) but activated only the ones you selected (in green). To view this list of devices, you will switch over to the Azure Portal, and open the IoT Hub you provisioned.

7. From the IoT Hub blade, select **IoT Devices** under Explorers on the left-hand menu.

   ![On the IoT Hub blade, in the Explorers section, under Explorers, IoT Devices is highlighted.](media/iot-hub-explorers-iot-devices.png 'IoT Hub blade, Explorers section')

8. You should see all 10 devices listed, with the ones that you activated having a status of **enabled**.

   ![Devices in the Device ID list have a status of either enabled or disabled.](media/iot-hub-iot-devices-list.png 'Device ID list')

9. Return to the **Smart Meter Simulator** window.

10. Select **Connect**. Within a few moments, you should begin to see activity as the windows change color indicating the smart meters are transmitting telemetry. The grid on the left will list each telemetry message transmitted and the simulated temperature value.

    ![On the Smart Meter Simulator, the Connect button is highlighted, and one of the green windows has now turned to blue. The current windows count is now seven gray, two green, and one blue.](media/smart-meter-simulator-connect.png 'Fabrikam Smart Meter Simulator')

11. Allow the smart meter to continue to run. (Whenever you want to stop the transmission of telemetry, select the **Disconnect** button.)

## Exercise 3: Hot path data processing with Stream Analytics

Duration: 45 minutes

Fabrikam would like to visualize the "hot" data showing the average temperature reported by each device over a 5-minute window in Power BI.

### Task 1: Create a Stream Analytics job for hot path processing to Power BI

1. In the [Azure Portal](https://portal.azure.com), select **+ Create a resource**, enter "stream analytics" into the Search the Marketplace box, select **Stream Analytics job** from the results, and select **Create**.

   ![In the Azure Portal, +Create a resource is highlighted, "stream analytics" is entered into the Search the Marketplace box, and Stream Analytics job is highlighted in the results.](media/create-resource-stream-analytics-job.png 'Create Stream Analytics job')

2. On the New Stream Analytics Job blade, enter the following:

   - **Job name**: Enter hot-stream.
   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group.
   - **Location**: Select the location you are using for resources in this hands-on lab.
   - **Hosting environment**: Select Cloud.
   - **Streaming units**: Change the value to 1 by sliding the slider all the way left.

     ![The New Stream Analytics Job blade is displayed, with the previously mentioned settings entered into the appropriate fields.](media/stream-analytics-job-create.png 'New Stream Analytics Job blade')

3. Select **Create**.

4. Once provisioned, navigate to your new Stream Analytics job in the portal.

5. On the Stream Analytics job blade, select **Inputs** from the left-hand menu, under Job Topology, then select **+Add stream input**, and select **IoT Hub** from the dropdown menu to add an input connected to your IoT Hub.

   ![On the Stream Analytics job blade, Inputs is selected under Job Topology in the left-hand menu, and +Add stream input is highlighted in the Inputs blade, and IoT Hub is highlighted in the drop down menu.](media/stream-analytics-job-inputs-add.png 'Add Stream Analytics job inputs')

6. On the New Input blade, enter the following:

   - **Input alias**: Enter temps.
   - Choose **Select IoT Hub from your subscriptions**.
   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **IoT Hub**: Select the smartmeter-hub-SUFFIX IoT Hub.
   - **Endpoint**: Select Messaging.
   - **Shared access policy name**: Select service.
   - **Consumer Group**: Leave set to \$Default.
   - **Event serialization format**: Select JSON.
   - **Encoding**: Select UTF-8.
   - **Event compression type**: Leave set to None.

     ![IoT Hub New Input blade is displayed with the values specified above entered into the appropriate fields.](media/stream-analytics-job-inputs-add-iot-hub-input.png 'IoT Hub New Input blade')

7. Select **Save**.

8. Next, select **Outputs** from the left-hand menu, under Job Topology, and select **+ Add**, then select **Power BI** from the drop-down menu.

   ![Outputs is highlighted in the left-hand menu, under Job Topology, +Add is selected, and Power BI is highlighted in the drop down menu.](media/stream-analytics-job-outputs-add-power-bi.png 'Add Power BI Output')

9. On the Power BI output blade, enter the following:

   - **Output alias**: Set to powerbi.

   - Select **Authorize** to authorize the connection to your Power BI account. When prompted in the popup window, enter the account credentials you used to create your Power BI account in [Before the hands-on lab setup guide, Task 1](./Before%20the%20HOL%20-%20Internet%20of%20Things.md).

     ![Power BI new output blade. Output alias is selected and contains powerbi. Authorize button is highlighted.](media/stream-analytics-job-outputs-add-power-bi-authorize.png 'Power BI new output blade')

   - For the remaining Power BI settings, enter the following:

     - **Group Workspace**: Select the default, My Workspace.
     - **Dataset Name**: Enter avgtemps.
     - **Table Name**: Enter avgtemps.

10. Select **Save**.

    ![Power BI blade. Output alias is powerbi, dataset name is avgtemps, table name is avgtemps.](media/stream-analytics-job-outputs-add-power-bi-save.png 'Add Power BI Output')

11. Next, select **Query** from the left-hand menu, under Job Topology.

    ![Under Job Topology, Query is selected.](./media/stream-analytics-job-query.png 'Stream Analytics Query')

12. In the query text box, paste the following query.

    ```sql
    SELECT AVG(temperature) AS Average, id
    INTO powerbi
    FROM temps
    GROUP BY TumblingWindow(minute, 5), id
    ```

13. Select **Save**, and **Yes** when prompted with the confirmation.

    ![Save button on the Query blade is highlighted](./media/stream-analytics-job-query-save.png 'Query Save button')

14. Return to the Overview blade on your Stream Analytics job and select **Start**.

    ![The Start button is highlighted on the Overview blade.](./media/stream-analytics-job-start.png 'Overview blade start button')

15. In the Start job blade, select **Now** (the job will start processing messages from the current point in time onward).

    ![Now is selected on the Start job blade.](./media/stream-analytics-job-start-job.png 'Start job blade')

16. Select **Start**.

17. Allow your Stream Analytics Job a few minutes to start.

18. Once the Stream Analytics Job has successfully started, verify that you are showing a non-zero amount of **Input Events** on the **Monitoring** chart on the overview blade. You may need to reconnect your devices on the Smart Meter Simulator and let it run for a while to see the events.

    ![The Stream Analytics job monitoring chart is diplayed with a non-zero amount of input events highlighted.](media/stream-analytics-job-monitoring-input-events.png 'Monitoring chart for Stream Analytics job')

### Task 2: Visualize hot data with Power BI

1. Sign in to your Power BI subscription (<https://app.powerbi.com>) to see if data is being collected.

2. Select **My Workspace** on the left-hand menu, then select the **Datasets tab**, and locate the **avgtemps** dataset from the list.

   ![On the Power BI window, My Workspace is highlighted in the left pane, and the Datasets tab is highlighted in the right pane, and the avgtemps dataset is highlighted.](media/power-bi-workspaces-datasets-avgtemps.png 'Power BI Datasets')

3. Select the Create Report button under the Actions column.

   ![On the Datasets tab, under Actions, the Create Report button is highlighted.](./media/power-bi-datasets-avgtemps-create-report.png 'Datasets tab, Action column')

4. On the Visualizations palette, select **Stacked column chart** to create a chart visualization.

   ![On the Visualizations palette, the stacked column chart icon is highlighted.](./media/power-bi-visualizations-stacked-column-chart.png 'Visualizations palette')

5. In the Fields listing, drag the **id** field, and drop it into the **Axis** field.

   ![Under Fields, an arrow points from the id field under avgtemps, to the same id field now located in the Visualizations listing, under Axis.](./media/power-bi-visualizations-stacked-column-chart-axis.png 'Visualizations and Fields')

6. Next, drag the **average** field and drop it into the **Value** field.

   ![Under Fields, an arrow points from the average field under avgtemps, to the same id field now located in the Visualizations listing, under Value.](./media/power-bi-visualizations-stacked-column-chart-value.png 'Visualizations and Fields')

7. Now, set the Value to **Max of average**, by clicking the down arrow next to **average**, and select **Maximum**.

   ![On the Value drop-down list, Maximum is highlighted.](./media/power-bi-visualizations-stacked-column-chart-value-maximum.png 'Value drop-down list')

8. Repeat steps 5-8, this time adding a Stacked Column Chart for **Min of average**. (You may need to click on any area of white space on the report designer surface to deselect the Max of average by id chart visualization.)

   ![Min of average is added under Value.](./media/power-bi-visualizations-stacked-column-chart-value-minimum.png 'Min of average')

9. Next, add a **table visualization**.

   ![On the Visualizations palette, the table icon is highlighted.](./media/power-bi-visualizations-table.png 'Visualizations pallete')

10. Set the values to **id** and **Average of average**, by dragging and dropping both fields in the Values field, then selecting the dropdown next to average, and selecting Average.

    ![ID and Average of average now display under Values.](./media/power-bi-visualizations-table-average-of-average.png 'Table Visualization values')

11. Save the report.

    ![Under File, Save is highlighted.](media/power-bi-save-report.png 'Save report')

12. Enter the name "Average Temperatures," and select **Save**.

    ![The report name is set to Average Temperatures.](./media/power-bi-save-report-average-temperatures.png 'Save your report')

13. Switch to **Reading View**.

    ![Power BI report Reading view button](media/power-bi-report-reading-view.png 'Reading view button')

14. Within the report, select one of the columns to see the data for just that device.

    ![The report window has two bar graphs: Max of average by id, and Min of average by id. both bar charts list data for Device0, Device1, Device3, Device8, and Device9. Device1 is selected. On the right, a table displays data for Device1, with an Average of average value of 69.50.](./media/power-bi-report-reading-view-single-device.png 'Report window')

## Exercise 4: Cold path data processing with Azure Databricks

Duration: 60 minutes

Fabrikam would like to be able to capture all the "cold" data into scalable storage so that they can summarize it periodically using a Spark SQL query.

### Task 1: Create a Storage account

1. In the [Azure portal](https://portal.azure.com), select **+ Create a resource**, enter "storage account" into the Search the Marketplace box, select **Storage account - blob, file, table, queue** from the results, and select **Create**.

   ![In the Azure portal, +Create a resource is highlighted in the navigation pane, "storage account" is entered into the Search the Marketplace box, and Storage account - blob, file, table, queue is highlighted in the results.](media/create-resource-storage-account.png 'Create Storage account')

2. In the Create storage account blade, enter the following:

   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group.
   - **Storage account name**: Enter smartmetersSUFFIX.
   - **Location**: Select the location you are using for resources in this hands-on lab.
   - **Performance**: Select Standard.
   - **Account kind**: Select Storage (general purpose v1).
   - **Replication**: Select Locally-redundant storage (LRS).

   ![The Create storage account blade is displayed, with the previously mentioned settings entered into the appropriate fields.](media/storage-account-create-new.png 'Create storage account')

3. Select **Next: Advanced >**.

4. In the Advanced tab, select the following:

   - **Secure transfer required**: Select Disabled.
   - **Virtual network**: Select None.

   ![The Create storage account blade is displayed with options under the Advanced tab.](media/storage-account-create-new-advanced.png 'Create storage account - Advanced')

5. Select **Review + create**.

6. In the Review tab, select **Create**.

7. Once provisioned, navigate to your storage account, select **Access keys** from the left-hand menu, and copy the key1 Key value into a text editor, such as Notepad, for later use.

### Task 2: Create the Stream Analytics job for cold path processing

To capture all metrics for the cold path, set up another Stream Analytics job that will write all events to Blob storage for analysis with Azure Databricks.

1. In the [Azure Portal](https://portal.azure.com), select **+ Create a resource**, enter "stream analytics" into the Search the Marketplace box, select **Stream Analytics job** from the results, and select **Create**.

   ![In the Azure Portal, +Create a resource is highlighted, "stream analytics" is entered into the Search the Marketplace box, and Stream Analytics job is highlighted in the results.](media/create-resource-stream-analytics-job.png 'Create Stream Analytics job')

2. On the New Stream Analytics Job blade, enter the following:

   - **Job name**: Enter cold-stream.
   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Resource group**: Select the hands-on-lab-SUFFIX resource group.
   - **Location**: Select the location you are using for resources in this hands-on lab.
   - **Hosting environment**: Select Cloud.
   - **Streaming units**: Drag the slider all the way to the left to select 1 streaming unit.

     ![The New Stream Analytics Job blade is displayed, with the previously mentioned settings entered into the appropriate fields.](media/stream-analytics-job-create-cold-stream.png 'New Stream Analytics Job blade')

3. Select **Create**.

4. Once provisioned, navigate to your new Stream Analytics job in the portal.

5. On the Stream Analytics job blade, select **Inputs** from the left-hand menu, under Job Topology, then select **+Add stream input**, and select **IoT Hub** from the dropdown menu to add an input connected to your IoT Hub.

   ![On the Stream Analytics job blade, Inputs is selected under Job Topology in the left-hand menu, and +Add stream input is highlighted in the Inputs blade, and IoT Hub is highlighted in the drop down menu.](media/stream-analytics-job-inputs-add.png 'Add Stream Analytics job inputs')

6. On the New Input blade, enter the following:

   - **Input alias**: Enter iothub.
   - Choose **Select IoT Hub from your subscriptions**.
   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **IoT Hub**: Select the smartmeter-hub-SUFFIX IoT Hub.
   - **Endpoint**: Select Messaging.
   - **Shared access policy name**: Select service.
   - **Consumer Group**: Leave set to \$Default.
   - **Event serialization format**: Select JSON.
   - **Encoding**: Select UTF-8.
   - **Event compression type**: Leave set to None.

     ![IoT Hub New Input blade is displayed with the values specified above entered into the appropriate fields.](media/stream-analytics-job-inputs-add-iot-hub-input-cold-stream.png 'IoT Hub New Input blade')

7. Select **Save**.

8. Next, select **Outputs** from the left-hand menu, under Job Topology, and select **+ Add**, then select **Blob storage** from the drop-down menu.

   ![Outputs is highlighted in the left-hand menu, under Job Topology, +Add is selected, and Blob storage is highlighted in the drop down menu.](media/stream-analytics-job-outputs-add-blob-storage.png 'Add Blob storage Output')

9. On the Blob storage output blade, enter the following:

   - **Output alias**: Set to blobs.
   - Choose **Select blob storage from your subscriptions**.
   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **Storage account**: Select the smartmetersSUFFIX storage account you created in the previous task.
   - **Container**: Choose Create new and enter smartmeters.
   - **Path pattern**: Enter smartmeters/{date}/{time}.
   - **Date format**: Select YYYY-DD-MM.
   - **Time format**: Select HH.
   - **Event serialization format**: Select CSV.
   - **Delimiter**: Select comma (,).
   - **Encoding**: Select UTF-8.

     ![Blob storage New output blade is displayed, with the values mentioned above entered into the appropriate fields.](media/stream-analytics-job-outputs-blob-storage-new.png 'Add Blob storage Output')

10. Select **Save**.

11. Next, select **Query** from the left-hand menu, under Job Topology.

    ![Under Job Topology, Query is selected.](./media/stream-analytics-job-query.png 'Stream Analytics Query')

12. In the query text box, paste the following query.

    ```sql
    SELECT
          *
    INTO
          blobs
    FROM
          iothub
    ```

13. Select **Save**, and **Yes** when prompted with the confirmation.

    ![Save button on the Query blade is highlighted](./media/stream-analytics-job-query-save.png 'Query Save button')

14. Return to the Overview blade on your Stream Analytics job and select **Start**.

    ![The Start button is highlighted on the Overview blade.](./media/stream-analytics-job-start.png 'Overview blade start button')

15. In the Start job blade, select **Now** (the job will start processing messages from the current point in time onward).

    ![Now is selected on the Start job blade.](./media/stream-analytics-job-start-job.png 'Start job blade')

16. Select **Start**.

17. Allow your Stream Analytics Job a few minutes to start.

18. Once the Stream Analytics Job has successfully started, verify that you are showing a non-zero amount of **Input Events** on the **Monitoring** chart on the overview blade. You may need to reconnect your devices on the Smart Meter Simulator and let it run for a while to see the events.

    ![The Stream Analytics job monitoring chart is diplayed with a non-zero amount of input events highlighted.](media/stream-analytics-job-monitoring-events.png 'Monitoring chart for Stream Analytics job')

### Task 3: Verify CSV files in blob storage

In this task, we are going to verify that the CSV file is being written to blob storage.

> **Note**: This can be done via Visual Studio, or Azure Storage Explore, or using the Azure portal. For this lab, we will perform the task using Azure Storage Explore. [Install Azure Storage Explore](https://azure.microsoft.com/en-us/features/storage-explorer/)

1. Run **Microsoft Azure Storage Explore**. In **Storage Explorer**, select Account Management, and connect to your Microsoft Azure Subscription.

   ![On the Visual Studio View menu, Cloud Explorer is highlighted.](./media/visual-studio-cloud-explorer-account-management_new.png 'Visual Studio View menu')

2. If prompted, sign into your Azure account.

3. Allow Storage Explorer about 30 seconds to load your subscription resources.

5. Expand your Azure account, then expand **Storage Accounts**, expand the smartmetersSUFFIX storage account, then right-click the smartmeters container, and select **Open**. It may take a few moments to load your storage accounts.

   ![Storage accounts is expanded in the Visual Studio Cloud Explorer, with the smartmetersSUFFIX account is expanded, and the Open menu item highlighted for the smartmeters container.](media/visual-studio-cloud-explorer-storage-accounts.png 'Cloud Explorer Storage Accounts')

6. Verify files are being written to Blob storage (the files should be located underneath the smartmeters container).

   ![Files are listed in the blob storage account, as written by the cold path route in IoT Hub Messaging.](media/smart-meters-cold-path-files.png 'Smart meters files in blob storage')

### Task 4: Process with Spark SQL

In this task, you will create a new Databricks notebook to perform some processing and visualization of the cold path data using Spark.

> **Note**: The complete Databricks notebook can be found in the Databricks-notebook folder of the GitHub repo associated with this hands-on lab, should you need to reference it for troubleshooting.

1. In the [Azure portal](https://portal.azure.com), navigate to the Azure Databricks workspace you created in the [Before the hands-on lab setup guide](./Before%20the%20HOL%20-%20Internet%20of%20Things.md) exercises, and select **Launch Workspace**.

   ![On the Azure Databricks Service blade, the Launch Workspace button is highlighted.](media/azure-databricks-launch-workspace.png 'Launch Azure Databricks Workspace')

2. On the Azure Databricks landing page, create a new notebook by selecting **Notebook** under New.

   ![Notebook is highlighted under New on the Azure Databricks landing page.](media/azure-databricks-new-notebook.png 'New Databricks Notebook')

3. In the Create Notebook dialog, enter **smartmeters** as the name, and select **Python** as the language, then select **Create**.

   ![In the Create Notebook dialog, smartmeters is entered as the Name, and Python is selected in the Language drop down.](media/azure-databricks-create-notebook-dialog.png 'Create Notebook dialog')

4. On the notebook page, attach the notebook to your cluster by selecting Detached, and then selecting your cluster under Attach to.

   ![In the notebook page, Detached is expanded, and the iot-cluster is highlighted under Attach to.](media/azure-databricks-notebook-attach.png 'Attach notebook to cluster')

5. If your cluster is stopped, you can select the down arrow next to your attached cluster name, and select Start Cluster from the menu, then select Confirm when prompted.

   ![The iot-cluster is attached and its menu is expanded, with Start Cluster highlighted](media/azure-databricks-notebook-start-cluster.png 'Start cluster')

6. In the first cell of your Databricks notebook (referred to as a paragraph in notebook jargon), enter the following **Python code** that creates widgets in the notebook for entering your Azure storage account name and key.

   ```python
   # Create widgets for storage account name and key
   dbutils.widgets.text("accountName", "", "Account Name")
   dbutils.widgets.text("accountKey", "", "Account Key")
   ```

7. Now, select the Run button on the right side of the cell, and select **Run cell**.

   ![A cell in a Databricks Notebook is displayed, and the Run menu is visible with Run Cell highlighted in the menu.](media/azure-databricks-notebook-run-cell.png 'Datebricks Notebook run cell')

8. When the cell finishes executing, you will see the Account Key and Account Name widgets appear at the top of the notebook, just below the toolbar.

   ![In the Databricks notebook, Account Key and Account Name widgets are highlighted.](media/azure-databricks-notebook-widgets.png 'Databricks Notebooks widgets')

9. You will also notice a message at the bottom of the cell indicating that the cell execution completed, and the amount of time it took.

   ![A message is displayed at the bottom of the cell indicating how long the command took to execute.](media/azure-databricks-cell-execution-time.png 'Cell execution time')

10. Enter your Azure Storage account key into the Account Key widget text box, and your Azure storage account name into the Account Name widget text box. These values can be obtained from the Access keys blade in your storage account.

    ![The Account Key and Account Name widgets are populated with values from the Azure storage account.](media/azure-databricks-notebook-widgets-populated.png 'Databricks Notebooks widgets')

11. At the bottom of the first cell, select the + button to insert a new cell below it.

    ![The Insert new cell button is highlighted at the bottom of the Databricks cell.](media/azure-databricks-insert-new-cell.png 'Insert new cell')

12. In the new cell, paste the following code that will assign the values you entered into the widgets you created above into variables that will be used throughout the notebook.

    ```python
    # Get values entered into widgets
    accountName = dbutils.widgets.get("accountName")
    accountKey = dbutils.widgets.get("accountKey")
    ```

13. Run the cell.

14. Insert a new cell into the notebook, and paste the following code to mount your blob storage account into Databricks File System (DBFS), then run the cell.

    ```python
    # Mount the blob storage account at /mnt/smartmeters. This assumes your container name is smartmeters, and you have a folder named smartmeters within that container, as specified in the exercises above.
    dbutils.fs.mount(
      source = "wasbs://smartmeters@" + accountName + ".blob.core.windows.net/smartmeters",
      mount_point = "/mnt/smartmeters",
      extra_configs = {"fs.azure.account.key." + accountName + ".blob.core.windows.net": accountKey})
    ```

    > **Note**: Mounting Azure Blob storage directly to DBFS allows you to access files as if they were on the local file system.

15. Once your blob storage account is mounted, you can access them with Databricks Utilities, `dbutils.fs` commands. Insert a new cell, and paste the code below to see how `dbutils.fs.ls` can be used to list the files and folders directly below the smartmeters folder.

    ```python
    # Inspect the file structure
    display(dbutils.fs.ls("/mnt/smartmeters/"))
    ```

16. Run the cell.

17. You know from inspecting the files in the storage container that the files are contained within a folder structure resembling, `smartmeters/YYYY-MM-DD/HH`. You can use wildcards to obfuscate the date and hour folders, as well as the file names, and access all the files in all the folders. Insert another cell into the notebook, paste the following code, and run the cell to load the data from the files in blob storage into a Databricks Dataframe.

    ```python
    # Create a Dataframe containing data from all the files in blob storage, regardless of the folder they are located within.
    df = spark.read.options(header='true', inferSchema='true').csv("dbfs:/mnt/smartmeters/*/*/*.csv",header=True)
    print(df.dtypes)
    ```

18. The cell above also outputs the value of the `df.dtypes` property, which is a list of the data types of the columns added to the Dataframe, similar to the following:

    ![Output from the df.dtypes property is displayed.](media/azure-databricks-df-dtypes-output.png 'Output from Dataframe dtypes')

19. Insert another cell, and run the following code to view the first 10 records contained in the Dataframe.

    ```python
    df.show(10)
    ```

20. Now, you can save the Dataframe to a global table in Databricks. This will make the table accessible to all users and clusters in your Databricks workspace. Insert a new cell, and run the following code.

    ```python
    df.write.mode("overwrite").saveAsTable("SmartMeters")
    ```

21. Now, you will use the `%sql` magic command to change the language of the next cell to SQL from the notebook's default language, Python, then execute a SQL command to aggregate the SmartMeter data by average temperature. Paste the following code into a new cell, and run the cell.

    ```sql
    %sql
    SELECT id, COUNT(*) AS count, AVG(temp) AS averageTemp FROM SmartMeters GROUP BY id ORDER BY id
    ```

22. The output from the SQL command should resemble the following table:

    ![Output from executing a SQL statement a Databricks notebook cell using the %sql magic command.](media/azure-databricks-notebook-sql-magic-command.png 'SQL magic command')

23. Now, execute the same command in a new cell, this time using Spark SQL so you can save the summary data into a Dataframe. Copy and execute the following code into a new cell.

    ```python
    # Query the table to create a Dataframe containing the summary
    summary = spark.sql("SELECT id, COUNT(*) AS count, AVG(temp) AS averageTemp FROM SmartMeters GROUP BY id ORDER BY id")

    # Save the new pre-computed table
    summary.write.mode("overwrite").saveAsTable("DeviceSummary")
    ```

24. Next, query from this summary table by executing the following query in a new cell.

    ```sql
    %sql
    SELECT * FROM DeviceSummary
    ```

25. Below the results table, button provide access to change the visualization for tabular output. Select the **chart** button, and then select **Plot Options**.

    ![Buttons for displaying tablular results in different formats in Databricks](media/azure-databricks-notebook-visualizations.png 'Visualization options')

26. In the Customize Plot dialog, set the following:

    - **Keys**: Add id.
    - **Values**: Add averageTemp.
    - **Aggregation**: Select AVG.
    - Select **Grouped** as the chart type.
    - **Display type**: Select Bar chart.

      ![Plot customization options dialog in Azure databricks, with id in the Keys field, averageTemp in the Values field, Aggregation set to AVG, and the chart set to a grouped bar chart.](media/azure-databricks-notebook-customize-plot.png)

27. Select **Apply**.

28. Observe the results graphed as a column chart, where each column represents a device's average temperature.

    ![A bar chart is displayed, with devices on the X axis, and average temperations on the Y axis.](media/azure-databricks-notebook-visualizations-bar-chart.png 'Bar chart')

## Exercise 5: Reporting device outages with IoT Hub Operations Monitoring

Duration: 20 minutes

Fabrikam would like to be alerted when devices disconnect and fail to reconnect after a period. Since they are already using PowerBI to visualize hot data, they would like to see a list of any of these devices in a report.

### Task 1: Enable verbose connection monitoring on the IoT Hub

To keep track of device connects and disconnects, we first need to enable verbose connection monitoring.

1. In the [Azure Portal](https://portal.azure.com), open the IoT Hub you provisioned earlier, **smartmeter-hub-SUFFIX**.

2. Under Settings in the left-hand menu, select **Operations monitoring**.

   ![Under Settings, Operations monitoring is selected.](media/iot-hub-settings-operations-monitoring.png 'Operations monitoring')

3. Select **Verbose** for **Log events when a device connects or disconnects from the IoT Hub**.

   ![In the Monitoring categories section, Verbose is selected under Log events when a device connects or disconnects from the IoT Hub.](./media/iot-hub-operation-monitoring-device-connections-verbose-logging.png 'Monitoring categories section')

4. Select **Save**.

### Task 2: Collect device connection telemetry with the hot path Stream Analytics job

Now that the device connections are being logged, update your hot path Stream Analytics job (the first one you created) with a new input that ingests device telemetry from Operations Monitoring. Next, create a query that joins all connected and disconnected events with a `DATEDIFF` function that only returns devices with a disconnect event, but no reconnect event within 120 seconds. Output the events to Power BI.

1. In the [Azure Portal](https://portal.azure.com), open the **hot-stream** Stream Analytics job (the first one you created).

2. Stop the job if it is currently running, from the Overview blade, by selecting **Stop**, then **Yes** when prompted.

   ![The Stop button is highlighted on the Stream Analytics job Overview blade.](media/stream-analytics-job-stop.png 'Stop Stream Analytics Job')

3. On the Stream Analytics job blade, select **Inputs** from the left-hand menu, under Job Topology, then select **+Add stream input**, and select **IoT Hub** from the dropdown menu to add an input connected to your IoT Hub.

   ![On the Stream Analytics job blade, Inputs is selected under Job Topology in the left-hand menu, and +Add stream input is highlighted in the Inputs blade, and IoT Hub is highlighted in the drop down menu.](media/stream-analytics-job-inputs-add.png 'Add Stream Analytics job inputs')

4. On the New Input blade, enter the following:

   - **Input alias**: Enter connections.
   - Choose **Select IoT Hub from your subscriptions**.
   - **Subscription**: Select the subscription you are using for this hands-on lab.
   - **IoT Hub**: Select the smartmeter-hub-SUFFIX IoT Hub.
   - **Endpoint**: Select Operations monitoring.
   - **Shared access policy name**: Select service.
   - **Consumer Group**: Leave set to \$Default.
   - **Event serialization format**: Select JSON.
   - **Encoding**: Select UTF-8.
   - **Event compression type**: Leave set to None.

     ![IoT Hub New Input blade is displayed with the values specified above entered into the appropriate fields.](media/stream-analytics-job-inputs-add-iot-hub-input-operations-management.png 'IoT Hub New Input blade')

5. Select **Save**.

6. Next, select **Outputs** from the left-hand menu, under Job Topology, and select **+ Add**, then select **Power BI** from the drop-down menu.

   ![Outputs is highlighted in the left-hand menu, under Job Topology, +Add is selected, and Power BI is highlighted in the drop down menu.](media/stream-analytics-job-outputs-add-power-bi.png 'Add Power BI Output')

7. On the Power BI output blade, enter the following:

   - **Output alias**: Set to powerbi-outage.
   - Select **Authorize** to authorize the connection to your Power BI account. When prompted in the popup window, enter the account credentials you used to create your Power BI account in Before the Hands-on Lab, Task 1.
   - **Group Workspace**: Select the default, My Workspace.
   - **Dataset Name**: Enter deviceoutage.
   - **Table Name**: Enter deviceoutage.

     ![Power BI blade. Output alias is powerbi-outage, dataset name is deviceoutage, table name is deviceoutage.](media/stream-analytics-job-outputs-add-power-bi-operations-management.png 'Add Power BI Output')

8. Select **Save**.

9. Next, select **Query** from the left-hand menu, under Job Topology.

   ![Under Job Topology, Query is selected.](./media/stream-analytics-job-query.png 'Stream Analytics Query')

10. Replace the hot path query, which selects the averages of the temperatures into the PowerBI output, with queries that perform the following:

    - Select **device disconnection events**.
    - Select **device connection events**.
    - Join these two streams together using the Stream Analytics `DATEDIFF` operation on the `LEFT JOIN`, and then filter out any records where there was a match. This gives us devices that had a disconnect event, but no corresponding connect event within 120 seconds. Output to the Service Bus.
    - Execute the original hot path query.

11. Replace the existing query with the following, and select **Save** in the **command bar** at the top. (Be sure to substitute in your output aliases and input aliases):

    ```sql
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

12. Select **Save**, and **Yes** when prompted with the confirmation.

    ![Save button on the Query blade is highlighted](./media/stream-analytics-job-query-save.png 'Query Save button')

13. Return to the Overview blade on your Stream Analytics job and select **Start**.

    ![The Start button is highlighted on the Overview blade.](./media/stream-analytics-job-start.png 'Overview blade start button')

14. In the Start job blade, select **Now** (the job will start processing messages from the current point in time onward).

    ![Now is selected on the Start job blade.](./media/stream-analytics-job-start-job.png 'Start job blade')

15. Select **Start**.

16. Allow your Stream Analytics Job a few minutes to start.

### Task 3: Test the device outage notifications

Register and activate a few devices on the Smart Meter Simulator, then connect them. Deactivate them without reconnecting in order for them to show up in the device outage report we will create in the next task.

1. Run the Smart Meter Simulator from Visual Studio.

2. Select **Register**.

   ![Register button](./media/smart-meter-simulator-register.png 'Register button')

3. Select 3 of the windows to highlight them.

   ![In the SmartMeter Simulator, three white windows display.](./media/smart-meter-simulator-window-select.png 'SmartMeter Simulator')

4. Select **Activate**.

   ![Activate button](./media/smart-meter-simulator-activate.png 'Activate button')

5. Select **Connect**.

   ![Connect button](./media/smart-meter-simulator-connect.png 'Connect button')

6. After a few seconds, select **Disconnect**.

   ![Disconnect button](media/smart-meter-simulator-disconnect.png 'Disconnect button')

7. Select **Unregister**.

   ![Unregister button](media/smart-meter-simulator-unregister.png 'Unregister button')

### Task 4: Visualize disconnected devices with Power BI

1. Log on to **Power BI** to see if data is being collected.

2. As done previously, select My Workspace on the left-hand menu, then select the Datasets tab. A new dataset should appear, named **deviceoutage**. (It is starred to indicate it is new) If you do not see the dataset, you may need to connect your devices on the Smart Meter Simulator, then disconnect and unregister them and wait up to 5 minutes.

   ![On the Power BI window, My Workspace is highlighted in the left pane, and the Datasets tab is highlighted in the right pane. Under Name, deviceoutage is highlighted.](./media/power-bi-datasets-deviceoutage.png 'Power BI window')

3. Select the **Create report** icon under actions for the dataset.

   ![Next to deviceoutage, the Create report icon is highlighted.](./media/power-bi-datasets-deviceoutage-create-report.png 'Create report icon')

4. Add a **Table visualization**.

   ![The table icon is highlighted on the Visualizations palette.](./media/power-bi-visualizations-table.png 'Visualizations palette')

5. Select the **deviceid** and **time** fields, which will automatically be added to the table.

   ![In the Fields listing, under deviceoutage, deviceid and time are both selected.](./media/power-bi-visualizations-table-fields.png 'Fields listing')

6. You should see the Device Id of each of the devices you connected, and then disconnected for more than 2 minutes.

   ![DeviceId and Time results display for Device one, Device five, and Device 8.](./media/power-pi-visualizations-table-output.png 'DeviceId and Time results')

7. Save the report as **Disconnected Devices**.

   ![Screenshot of the Save this report option.](./media/power-bi-save-report.png 'Save this report option')

8. Switch to **Reading View**.

   ![Reading View button](./media/power-bi-report-reading-view.png 'Reading View button')

9. Within the report, select the column headers to sort by device or date. You may run a few more tests with the Smart Meter Simulator and periodically refresh the report to see new devices.

   ![This report has two columns: DeviceID, and Time. Data is sorted by time.](./media/power-bi-report-reading-view-column-sorting.png 'Report')

## After the hands-on lab

Duration: 10 mins

In this exercise, you will delete any Azure resources that were created in support of the lab. You should follow all steps provided after attending the Hands-on lab to ensure your account does not continue to be charged for lab resources.

### Task 1: Delete the resource group

1. Using the [Azure portal](https://portal.azure.com), navigate to the Resource group you used throughout this hands-on lab by selecting Resource groups in the left menu.

2. Search for the name of your research group, and select it from the list.

3. Select Delete in the command bar, and confirm the deletion by re-typing the Resource group name, and selecting Delete.

You should follow all steps provided _after_ attending the Hands-on lab.
