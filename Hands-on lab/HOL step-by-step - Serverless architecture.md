![](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png "Microsoft Cloud Workshops")

<div class="MCWHeader1">
Serverless architecture
</div>

<div class="MCWHeader2">
Hands-on lab step-by-step
</div>


Information in this document, including URL and other Internet Web site references, is subject to change without notice. Unless otherwise noted, the example companies, organizations, products, domain names, e-mail addresses, logos, people, places, and events depicted herein are fictitious, and no association with any real company, organization, product, domain name, e-mail address, logo, person, place or event is intended or should be inferred. Complying with all applicable copyright laws is the responsibility of the user. Without limiting the rights under copyright, no part of this document may be reproduced, stored in or introduced into a retrieval system, or transmitted in any form or by any means (electronic, mechanical, photocopying, recording, or otherwise), or for any purpose, without the express written permission of Microsoft Corporation.

Microsoft may have patents, patent applications, trademarks, copyrights, or other intellectual property rights covering subject matter in this document. Except as expressly provided in any written license agreement from Microsoft, the furnishing of this document does not give you any license to these patents, trademarks, copyrights, or other intellectual property.

The names of manufacturers, products, or URLs are provided for informational purposes only and Microsoft makes no representations and warranties, either expressed, implied, or statutory, regarding these manufacturers or the use of the products with any Microsoft technologies. The inclusion of a manufacturer or product does not imply endorsement of Microsoft of the manufacturer or product. Links may be provided to third party sites. Such sites are not under the control of Microsoft and Microsoft is not responsible for the contents of any linked site or any link contained in a linked site, or any changes or updates to such sites. Microsoft is not responsible for webcasting or any other form of transmission received from any linked site. Microsoft is providing these links to you only as a convenience, and the inclusion of any link does not imply endorsement of Microsoft of the site or the products contained therein.

© 2018 Microsoft Corporation. All rights reserved.

Microsoft and the trademarks listed at <https://www.microsoft.com/legal/intellectualproperty/Trademarks/Usage/General.aspx> are trademarks of the Microsoft group of companies. All other trademarks are property of their respective owners.

**Contents**

<!-- TOC -->

- [Serverles architecture hands-on lab step-by-step](#serverles-architecture-hands-on-lab-step-by-step)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Solution architecture](#solution-architecture)
  - [Exercise 1: Create functions in the portal](#exercise-1-create-functions-in-the-portal)
    - [Help references](#help-references)
    - [Task 1: Create function to save sensor data to Azure Cosmos DB](#task-1-create-function-to-save-sensor-data-to-azure-cosmos-db)
    - [Task 2: Add an Azure Cosmos DB output to the SaveSensorInfo function](#task-2-add-an-azure-cosmos-db-output-to-the-SaveSensorInfo-function)
  - [Exercise 2: Monitor your functions with Application Insights](#exercise-2-monitor-your-functions-with-application-insights)
    - [Help references](#help-references-1)
    - [Task 1: Provision an Application Insights instance](#task-1-provision-an-application-insights-instance)
    - [Task 2: Enable Application Insights integration in your Function Apps](#task-2-enable-application-insights-integration-in-your-function-apps)
    - [Task 3: Use the Live Metrics Stream to monitor functions in real time](#task-3-use-the-live-metrics-stream-to-monitor-functions-in-real-time)
    - [Task 4: Observe your functions dynamically scaling when resource-constrained](#task-4-observe-your-functions-dynamically-scaling-when-resource-constrained)
  - [Exercise 3: Explore your data in Azure Cosmos DB](#exercise-3-explore-your-data-in-azure-cosmos-db)
    - [Help references](#help-references-2)
    - [Task 1: Use the Azure Cosmos DB Data Explorer](#task-1-use-the-azure-cosmos-db-data-explorer)
  

<!-- /TOC -->

# Serverles architecture hands-on lab step-by-step 

## Abstract and learning objectives

In this hand-on lab, you will be challenged to implement an end-to-end scenario using a supplied sample that is based on Microsoft Azure Functions, Azure Cosmos DB, IoT Hub, and related services. The scenario will include implementing compute, storage, workflows, and monitoring, using various components of Microsoft Azure. The hands-on lab can be implemented on your own, but it is highly recommended to pair up with other members at the lab to model a real-world experience and to allow each member to share their expertise for the overall solution.

At the end of the hands-on-lab, you will have confidence in designing, developing, and monitoring a serverless solution that is resilient, scalable, and cost-effective.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

![The Solution diagram is described in the text following this diagram.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image2_New.png 'Solution diagram')



## Exercise 1: Create functions in the portal


Create two new Azure Functions written in Node.js, using the Azure portal. These will be triggered by Event Grid and output to Azure Cosmos DB to save the results of license plate processing done by the ProcessImage function.

### Help references

|                                                                   |                                                                                                               |
| ----------------------------------------------------------------- | :-----------------------------------------------------------------------------------------------------------: |
| **Description**                                                   |                                                   **Links**                                                   |
| Create your first function in the Azure portal                    |        <https://docs.microsoft.com/azure/azure-functions/functions-create-first-azure-function>         |
| Store unstructured data using Azure Functions and Azure Cosmos DB | <https://docs.microsoft.com/azure/azure-functions/functions-integrate-store-unstructured-data-cosmosdb> |

### Task 1: Create function to save sensor  data to Azure Cosmos DB

In this task, you will create a new Node.js function triggered by Event Grid and that outputs successfully save sensor data to Azure Cosmos DB.

1.  Using a new tab or instance of your browser navigate to the Azure Management portal, <http://portal.azure.com>.

2.  Open the **hands-on-lab-SUFFIX** resource group, then select the Azure Function App you created whose name starts with **IoTFunction**. If you did not use this naming convention, make sure you select the Function App that you [did not]{.underline} deploy to in the previous exercise.

3.  Select **Functions** in the menu. In the **Functions** blade, select **+ New Function**.

    ![In the IoTFunction-wy blade, in the pane under Function Apps, IoTFunction-wy is expanded, and Functions is selected. In the pane, the + New function button is selected.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image43_New.png 'TollBoothEvents2 blade')

4.  In the **Quickstart** dialog, select **In-portal** then select **Continue**

5.  Select **More templates**, then select **Finish and view templates**

6.  select the **IoT Hub(Event Hub)** template.  

    a.  If prompted, click **Install** and wait for the extension to install.

    b.  Click **Continue**

    ![In the Template search form, event grid is typed in the search field. Below, the Event Grid trigger function option displays.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image44_New.png 'Template search form')

7.  In the New Function form, fill out the following properties:

    a. For name, enter **SaveSensorInfo**

    ![In the New Function form SaveSensorInfo is typed in the Name field.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image45_New.png 'Event Grid trigger, New Function form')

    b. For Event Hub connection, click **new** and config IoT Hub, choose the IoT Hub starts with **smartmeter** and use **Events** under the Endpoint. then click **Select** button

     ![In the Connection select IoT Hub.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image45_New2.png 'Event Grid trigger, Connection Form')

    b.  Then select **Create**.

7)  Replace the code in the new SaveSensorInfo function with the following:

**Java Script**
```

   module.exports = function (context, IoTHubMessages) {
    context.log(`JavaScript eventhub trigger function called for message array: ${IoTHubMessages}`);

    var count = 0;
    var totalTemperature = 0.0;
    var temperature = 0.0;
    var deviceId = "";

    IoTHubMessages.forEach(message => {
        context.log(`Processed message: ${message}`);
        count++;
        temperature = message.temp;
        deviceId = message.id;
    });

    var output = {
        "deviceId": deviceId,
        "temperature": temperature
    };

    context.log(`Output content: ${output}`);

    context.bindings.outputDocument = output;

    context.done();
};

```

**C#**
```
#r "Newtonsoft.Json"

using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static IActionResult Run(string myIoTHubMessage, out object outputDocument, ILogger log)
{
    log.LogInformation($"C# IoT Hub trigger function processed a message: {myIoTHubMessage}");
	
    string deviceId="";
	float temp =0F;
    
    var raw_obj=JObject.Parse(myIoTHubMessage);
    deviceId=(string)raw_obj["id"];
    temp=(float)raw_obj["temp"];
   
	outputDocument = new
	{
		deviceId,
		temp
	};

    log.LogInformation($"output doc: {outputDocument}");

	return (ActionResult)new OkResult();
}
```


8.  Select **Save**.

### Task 2: Add an Azure Cosmos DB output to the SaveSensorInfo function

In this task, you will add an Azure Cosmos DB output binding to the SaveSensorInfo function, enabling it to save its data to the Processed collection.

1.  Expand the **SaveSensorInfo** function in the menu, the select **Integrate**.

2.  Under Outputs, select **+ New Output**, select **Azure Cosmos DB** from the list of outputs, then select **Select**.

    ![In the SaveSensorInfo blade, in the pane under Function Apps, SaveSensorInfo are expanded, and Integrate is selected. In the pane, + New Output is selected under Outputs. In the list of outputs, the Azure Cosmos DB tile is selected.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image48_New.png 'SaveSensorInfo blade')

3.  In the Azure Cosmos DB output form, select **new** next to the Azure Cosmos DB account connection field.

    ![The New button is selected next to the Azure Cosmos DB account connection field.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image49.png 'New button')

    > **Note**: If you see a notice for "Extensions not installed", click **Install**.

4.  Select your Cosmos DB account from the list that appears.

5.  Specify the following configuration options in the Azure Cosmos DB output form:

    a. For database name, type **IoTDB**.

    b. For the collection name, type **IoTCollection**.

    ![Under Azure Cosmos DB output the following field values display: Document parameter name, outputDocument; Collection name, Processed; Database name, LicensePlates; Azure Cosmos DB account conection, tollbooths_DOCUMENTDB.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image50_New.png 'Azure Cosmos DB output section')

6.  Select **Save**.

    > **Note**: you should wait for the template dependency to install if you were promted earlier.

## Exercise 2: Monitor your functions with Application Insights

Application Insights can be integrated with Azure Function Apps to provide robust monitoring for your functions. In this exercise, you will provision a new Application Insights account and configure your Function Apps to send telemetry to it.

### Help references

|                                                               |                                                                                        |
| ------------------------------------------------------------- | :------------------------------------------------------------------------------------: |
| **Description**                                               |                                       **Links**                                        |
| Monitor Azure Functions using Application Insights            |     <https://docs.microsoft.com/azure/azure-functions/functions-monitoring>      |
| Live Metrics Stream: Monitor & Diagnose with 1-second latency | <https://docs.microsoft.com/azure/application-insights/app-insights-live-stream> |

### Task 1: Use the Live Metrics Stream to monitor functions in real time

1.  Go to **LabVM** and launch the **Smart MeterSimulator** and make it send events to IoT Hub

2.  Open the Azure Function App you created whose name starts with **IoTFunction**, or the name you specified for the Function App containing the SaveSensorInfo function.

3.  Select **Application Insights** on the Overview pane.

    >**Note**:  It may take a few minutes for this link to display.

    ![In the IoTFunctionApp blade, under Configured features, the Application Insights link is selected.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image64_New.png 'TollBoothFunctionApp blade')

4.  In Application Insights, select **Live Metrics Stream** underneath Investigate in the menu.

    ![In the IoTFunctionMonitor blade, in the pane under Investigate, Live Metrics Stream is selected. ](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image65.png 'TollBoothMonitor blade')

5. On your browser window within Application Insights. You should start seeing new telemetry arrive, showing the number of servers online, the incoming request rate, CPU process amount, etc. You can select some of the sample telemetry in the list to the side to view output data.

    ![The Live Metrics Stream window displays information for the two online servers. Displaying line and point graphs include incoming requests, outgoing requests, and overvall health. To the side is a list of Sample Telemetry information. ](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image70_New.png 'Live Metrics Stream window')


## Exercise 3: Explore your data in Azure Cosmos DB

In this exercise, you will use the Azure Cosmos DB Data Explorer in the portal to view saved license plate data.

### Help references

|                       |                                                                 |
| --------------------- | :-------------------------------------------------------------: |
| **Description**       |                            **Links**                            |
| About Azure Cosmos DB | <https://docs.microsoft.com/azure/cosmos-db/introduction> |

### Task 1: Use the Azure Cosmos DB Data Explorer

1.  Open your Azure Cosmos DB account by opening the **hands-on-lab-SUFFIX** resource group, and then selecting the **Azure Cosmos DB account** name.

2.  Select **Data Explorer** from the menu.

    ![In the IoTDB - Data Explorer blade, Data Explorer is selected.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image75_New.png 'IoTDB - Data Explorer blade')

3.  Expand the **IoTCollection** collection, then select **Documents**. This will list each of the JSON documents added to the collection.

4.  Select one of the documents to view its contents. The first four properties are ones that were added by your functions. The remaining properties are standard and are assigned by Cosmos DB.

    ![Under Collections, IoTCollection is expanded, and Documents is selected. On the Documents tab, a document is selected, and to the side, the first four properties of the document (fileName, licencePlateText, timeStamp, and exported) are circled.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image76_New.png 'IoTDB - Data Explorer blade')

7.  Right-click on the **IoTCollection** collection and select **New SQL Query**.

    ![Under Collections, LicencePlates is expanded, and IoTCollection is selected. From its right-click menu, New SQL Query is selected.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image78.png 'Tollbooth - Data Explorer blade')

8.  Modify the SQL query to count the number of IoTCollection documents generated by Device0:

```
SELECT VALUE count(c.id) FROM c where c.deviceId = "Device0"
```

9.  Execute the query and observe the results. In our case, we have 693  documents that generated by Device0.

    ![On the Query 1 tab, under Execute Query, the previously defined SQL query displays. Under Results, the number 1369 is circled.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image79_New.png 'Query 1 tab')


