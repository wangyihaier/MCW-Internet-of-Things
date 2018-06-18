# Internet of Things hands-on lab

## Contents

* [Abstract](#abstract)
* [Overview](#overview)
* [Solution architecture](#solution-architecture)
* [Requirements](#requirements)
* [Before the hands-on lab](#before-the-hands-on-lab)
* [Hands-on lab](#hands-on-lab)

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

## Before the hands-on lab

Before attending the hands-on lab workshop, you should set up your environment for use in the rest of the hands-on lab.

You should follow all the steps provided in the [Before the hands-on lab](./Setup.md) section to prepare your environment before attending the hands-on lab. Failure to complete the Before the hands-on lab setup may result in an inability to complete the lab with in the time allowed.

## Hands-on lab

Select the guide you are using to complete the Hands-on lab below.

* [Step-by-step guide](./HOL-step-by-step-Intelligent-vending-machines.md)
  * Provides detailed, step-by-step instructions for completing the lab.
* [Unguided](./HOL-uguided-Intelligent-vending-machines.md)
  * This guide provides minimal instruction, and assumes a high-level of knowledge about the technologies used in this lab. This should typically only be used if you are doing this as part of a group.