![](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png 'Microsoft Cloud Workshops')

<div class="MCWHeader1">
Internet of Things
</div>

<div class="MCWHeader2">
Before the hands-on lab setup guide
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

<!-- TOC -->

- [Capture Device Events using Event Grid and Notify hands-on lab setup guide](#Capture-Device-Events using Event Grid-and-Notify-hands-on-lab-setup-guide)
  - [Overview](#Overview)
  - [Learning Objectives](#Learning-Objectives)
  - [Solution architecture](#Solution-architecture)
  - [Exercise](#Exercise)
       - [Create Logic App](#Create-Logic-App)
       - [Setup Notification by Sending Email](#Setup-Notification-by-Sending-Email)
       - [Copy Request URL](#Copy-Request-URL)
       - [Integrate With IoTHub](#Integrate-With-IoTHub)
       - [Add Device and Test Notification](#Add-Device-and-Test-Notification)
       - [Delete Device and Test Notification](#Delete-Device-and-Test-Notification)

<!-- /TOC -->

# Capture Device Events using Event Grid and Notify hands-on lab setup guide

## Overview
![Header Image](EventGrid/images/eventgrid.jpg)

Azure IoT Hub integrates with Azure Event Grid so that you can send event notifications to other services and trigger downstream processes. Configure your business applications to listen for IoT Hub events so that you can react to critical events in a reliable, scalable, and secure manner. For example, build an application to perform multiple actions like updating a database, creating a ticket, and delivering an email notification every time a new IoT device is registered to your IoT hub.

<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-IoT-Hub-Integration-with-Azure-Event-Grid/player" width="480" height="270" allowFullScreen frameBorder="0"></iframe>


## Learning Objectives

In this lab you will learn how to

* Create logic app to be able to send email notifications

* Create Event Grid

* Connect IoT Hub to Event Grid

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

![The Solution diagram is described in the text following this diagram.](images/Hands-onlabstep-by-step-Serverlessarchitectureimages/media/image2_New2.png 'Solution diagram')


## Exercise
### Create Logic App

Create a Logic App to be able to send email notifications

Click on **Create a resource**

![Create Resource](EventGrid/images/create_resource.png)

Click on **Enterprise Integration**

![Enterprise Integration](EventGrid/images/enterprise_integration.png)

Click on **Logic Apps**

![Create Logic App](EventGrid/images/logic_app.png)

Use existing resource group created in previous steps and press Create

![Create Logic App](EventGrid/images/02_Create_LogicApp_Submit.png)

Using Logic App Designer, Create New App

![Create App](EventGrid/images/03_Logic_App_designer.png)

Select HTTP Request

![Select HTTP Request](EventGrid/images/04_Http_Request.png)

Provide a Sample Payload

```code
[{
  "id": "56afc886-767b-d359-d59e-0da7877166b2",
  "topic": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/<Resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<IoT hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceCreated",
  "eventTime": "2018-01-02T19:17:44.4383997Z",
  "data": {
    "twin": {
      "deviceId": "LogicAppTestDevice",
      "etag": "AAAAAAAAAAE=",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        },
        "reported": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        }
      }
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice",
    "operationTimestamp": "2018-01-02T19:17:44.4383997Z",
    "opType": "DeviceCreated"
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

![Provide Sample Payload](EventGrid/images/05_Sample_Payload.png)

### Setup Notification by Sending Email 

Click on New Step

![New Step](EventGrid/images/06_New_Step.png)

Add an action

![Add an Action](EventGrid/images/07_Add_new_Action.png)

Choose Mail

![Choose Mail](EventGrid/images/08_Choose_Mail.png)

Finish Mail Actions

![Finish Mail Actions](EventGrid/images/09_send_email.png)

Sign in to email

![Sign in to email](EventGrid/images/10_signin_to_email.png)

Create Email template

![Create email template](EventGrid/images/11_Send_Email.png)

### Copy Request URL

![Copy Request URL](EventGrid/images/12_eventurl.png)

### Integrate With IoTHub

Integrate Logic App with IoTHub via Event Grid

![Imported Script](EventGrid/images/13_IoTHub_EventHub_New_1.png "Integrated with IoTHub")

Click on Event Subscription

![Integrated with IoTHub](EventGrid/images/14_empty_event_subscription_New.png "")

Copy the URL from previous steps into Subscriber Endpoint and click create

![Integrated with IoTHub](EventGrid/images/15_device_events_New.png)

![Complete Integration with IoTHub](EventGrid/images/15_device_events_New2.png)

### Add Device and Test Notification

Go To IoTHub -> IoT Devices (Device Management) -> Add

![Add Device](EventGrid/images/16_add_device.png)

Click Save button to create a new device

![Add Device](EventGrid/images/17_add_device.png)

You Should get an email notification

![Email Notification](EventGrid/images/18_email_generated_New.png)

### Delete Device and Test Notification

Go To IoTHub -> IoT Devices (Device Management) -> Select Device you created in previous step -> Delete

![Delete Device](EventGrid/images/19_delete_device.png)

You Should get an email notification

![Email Notification](EventGrid/images/20_email_generated_new.png)