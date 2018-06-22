# Internet of Things hands-on lab

## Contents

- [Abstract](#abstract)
- [Overview](#overview)
- [Solution architecture](#solution-architecture)
- [Requirements](#requirements)
- [Before the hands-on lab](#before-the-hands-on-lab)
- [Hands-on lab](#hands-on-lab)

## Abstract

In this hands-on lab, you will construct an end-to-end IoT solution simulating high velocity data emitted from smart meters and analyzed in Azure. You will design a lambda architecture, filtering a subset of the telemetry data for real-time visualization on the hot path, and storing all the data in long-term storage for the cold path.

At the end of this hands-on lab, you will be better able to build an IoT solution implementing device registration with the IoT Hub Device Provisioning Service and visualizing hot data with Power BI.

## Overview

Fabrikam provides services and smart meters for enterprise energy (electrical power) management. Their "*You-Left-The-Light-On*" service enables the enterprise to understand their energy consumption.

## Solution architecture

Below is a diagram of the solution architecture you will build in this lab. Please study this carefully, so you understand the whole of the solution as you are working on the various components.

![Diagram of the preferred solution. From a high-level, the commerce solution uses an API App to host the Payments web service with which the Vending Machine interacts to conduct purchase transactions. The Payment Web API invokes a 3rd party payment gateway as needed for authorizing and capturing credit card payments, and logs the purchase transaction to SQL DB. The data for these purchase transactions is stored using an In-Memory table with a Columnar Index, which will support the write-heavy workload while still allowing analytics to operate, such as queries coming from Power BI Desktop.](./media/preferred-solution-architecture.png "Preferred high-level architecture")

Messages are ingested from the Smart Meters via IoT Hub and temporarily stored there. A Stream Analytics job pulls telemetry messages from IoT Hub and sends the messages to two different destinations. There are two Stream Analytics jobs, one that retrieves all messages and sends them to Blob Storage (the cold path), and another that selects out only the important events needed for reporting in real time (the hot path). Data entering the hot path will be reported on using Power BI visualizations and reports. For the cold path, Azure Databricks can be used to apply the batch computation needed for the reports at scale.

Other alternatives for processing of the ingested telemetry would be to use an HDInsight Storm cluster, a WebJob running the EventProcessorHost in place of Stream Analytics, or HDInsight running with Spark streaming. Depending on the type of message filtering being conducted for hot and cold stream separation, IoT Hub Message Routing might also be used, but this has the limitation that messages follow a single path, so with the current implementation, it would not be possible to send all messages to the cold path, while simultaneously sending some of the same messages into the hot path. An important limitation to keep in mind for Stream Analytics is that it is very restrictive on the format of the input data it can process: the payload must be UTF8 encoded JSON, UTF8 encoded CSV (fields delimited by commas, spaces, tabs, or vertical pipes), or AVRO, and it must be well formed. If any devices transmitting telemetry cannot generate output in these formats (e.g., because they are legacy devices), or their output can be not well formed at times, then alternatives that can better deal with these situations should be investigated. Additionally, any custom code or logic cannot be embedded with Stream Analytics---if greater extensibility is required, the alternatives should be considered.

> NOTE: The preferred solution is only one of many possible, viable approaches.

## Requirements

- Microsoft Azure subscription must be pay-as-you-go or MSDN
  - Trial subscriptions will not work
- A virtual machine configured with:
  - Visual Studio Community 2017 15.6 or later
  - Azure SDK 2.9 or later (Included with Visual Studio 2017)
- A running Azure Databricks cluster (see [Before the hands-on lab](#before-the-hands-on-lab))

## Before the hands-on lab

Before attending the hands-on lab workshop, you should set up your environment for use in the rest of the hands-on lab.

You should follow all the steps provided in the [Before the hands-on lab](./Setup.md) section to prepare your environment before attending the hands-on lab. Failure to complete the Before the hands-on lab setup may result in an inability to complete the lab with in the time allowed.

## Hands-on lab

Select the guide you are using to complete the Hands-on lab below.

- [Step-by-step guide](./HOL-step-by-step-Intelligent-vending-machines.md)
  - Provides detailed, step-by-step instructions for completing the lab.
- [Unguided](./HOL-uguided-Intelligent-vending-machines.md)
  - This guide provides minimal instruction, and assumes a high-level of knowledge about the technologies used in this lab. This should typically only be used if you are doing this as part of a group.