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

- [Internet of Things before the hands-on lab setup guide](#internet-of-things-before-the-hands-on-lab-setup-guide)
  - [Requirements](#requirements)
  - [Before the hands-on lab](#before-the-hands-on-lab)
       - [Task 1: Provision a resource group](#task-1-provision-a-resource-group)
       - [Task 2: Set up a lab virtual machine](#task-2-set-up-a-lab-virtual-machine)
       - [Task 3: Provision Azure Databricks](#task-3-provision-azure-databricks)
       - [Task 4: Create Databricks cluster](#task-4-create-databricks-cluster)
       - [Task 5: Provision Power BI](#task-5-provision-power-bi)
       - [Task 6: Connect to your Lab VM](#task-6-connect-to-your-lab-vm)
       - [Task 7: Download Smart Meter Simulator project](#task-8-download-smart-meter-simulator-project)

<!-- /TOC -->

# Internet of Things before the hands-on lab setup guide

## Requirements

- Microsoft Azure subscription must be pay-as-you-go or MSDN.
  - Trial subscriptions will not work.
- A virtual machine configured with:
  - Visual Studio Community 2017 15.6 or later
  - Azure SDK 2.9 or later (Included with Visual Studio 2017)
- A running Azure Databricks cluster.

## Before the hands-on lab

Duration: 45 minutes

In the Before the hands-on lab exercise, you will set up your environment for use in the rest of the hands-on lab. You should follow all the steps provided in the Before the hands-on lab section to prepare your environment **before attending** the hands-on lab. Failure to do so will significantly impact your ability to complete the lab within the time allowed.

> **IMPORTANT**: Most Azure resources require unique names. Throughout this lab you will see the word “SUFFIX” as part of resource names. You should replace this with your Microsoft alias, initials, or another value to ensure the resource is uniquely named.

### Task 1: Provision a resource group

In this task, you will create an Azure resource group for the resources used throughout this lab.

1. In the [Azure portal](https://portal.azure.com), select **Resource groups**, select **+Add**, then enter the following in the Create an empty resource group blade:

   - **Name**: Enter hands-on-lab-SUFFIX.

   - **Subscription**: Select the subscription you are using for this hands-on lab.

   - **Resource group location**: Select the region you would like to use for resources in this hands-on lab. Remember this location so you can use it for the other resources you'll provision throughout this lab.

     ![Add Resource group Resource groups is highlighted in the navigation pane of the Azure portal, +Add is highlighted in the Resource groups blade, and "hands-on-labs" is entered into the Resource group name box on the Create an empty resource group blade.](media/create-resource-group.png 'Create resource group')

   - Select **Create**.

### Task 2: Setup a lab virtual machine

In this task, you will provision a virtual machine running Visual Studio Community 2017 that you will use as your development machine for this hands-on lab.

1. In the [Azure portal](https://portal.azure.com/), select **+Create a resource**, enter "visual studio community" into the Search the Marketplace box, select **Visual Studio Community 2017 (latest release) on Windows Server 2016 (x64)** from the results, and select **Create**.

   ![+Create a resource is selected in the Azure navigation pane, and "visual studio community" is entered into the Search the Marketplace box. Visual Studio Community 2017 (latest release) on Windows Server 2016 (x64) is selected in the results.](./media/create-resource-visual-studio-on-windows-server-2016.png 'Create Windows Server 2016 with Visual Studio Community 2017')

2. Set the following configuration on the Basics tab:

   - **Subscription**: Select the same subscription you are using for this hands-on lab.

   - **Resource Group**: Select Use existing, and select the hands-on-lab-SUFFIX resource group.

   - **Virtual machine name**: Enter LabVM

   - **Region**: Select the same region you selected for the resource group.

   - **Availability options**: Select No infrastructure redundancy required.

   - **Image**: The Visual Studio Community image you selected in the previous step should be listed here.

   - **Size**: Select a Standard D2 v3 size if it is not already selected.

   - **Username**: Enter demouser

   - **Password**: Enter Password.1!!

   - **Public inbound ports**: Select Allow selected ports.

   - **Selected inbound ports**: Select **RDP (3389)** from the drop down.

     ![Screenshot of the Create virtual machine blade, with fields set to the previously mentioned settings.](media/virtual-machine.png 'Create virtual machine blade')

   - Select **Review + create** to move to the next step.

3. Select **Create** on the Create blade to provision the virtual machine.

4. It may take 10+ minutes for the virtual machine to complete provisioning.

5. You can move on to the next task while waiting for the lab VM to provision.

### Task 3: Provision Azure Databricks

In this task, you will create an Azure Databricks workspace.

1. In the [Azure portal](https://portal.azure.com), select **+Create a resource**, enter "databricks" into the Search the Marketplace box, select **Azure Databricks** from the results, and select **Create**.

   ![In the Azure navigation pane, +Create a resource is selected. In the Everything blade, "databricks" is entered into the Search the Marketplace box, and Azure Databricks is selected in the results.](media/create-resource-azure-databricks.png 'Create Azure Databricks')

2. On the Azure Databricks Service blade, enter the following:

   - **Workspace name**: Enter iot-db-workspace-SUFFIX.

   - **Subscription**: Select the subscription you are using for this hands-on lab.

   - **Resource group**: Choose Use existing and select the hands-on-lab-SUFFIX resource group.

   - **Location**: Select the location you are using for resources in this hands-on lab.

   - **Pricing tier**: Select Standard.

     ![The Azure Databricks Service blade is displayed, with the values specified above entered into the appropriate fields.](media/azure-databricks-create-workspace.png 'Create Azure Databricks workspace')

   - Select **Create**.

### Task 4: Create Databricks cluster

In this task, you will create an Azure Databricks cluster within the workspace you created previously.

1. Once the deployment of the Databricks workspace is complete, select **Go to resource** on the notification you receive.

   ![Under Notifications in Azure, a message that the Azure Databricks deployment succeeded is displayed, and the Go to resource button is highlighted.](media/azure-databricks-resource-created.png 'Azure Databricks deployment succeeded')

2. On the Azure Databricks Service overview blade, select **Launch Workspace**.

   ![On the Azure Databricks Service blade, the Launch Workspace button is highlighted.](media/azure-databricks-launch-workspace.png 'Launch Azure Databricks Workspace')

3. In the new browser window that opens, select **Clusters** from the left-hand navigation menu, then select **+Create Cluster**.

   ![In the Azure Databricks workspace, Clusters in highlighted in the left-hand navigation menu, and the +Create Cluster button is highlighted.](media/azure-databricks-clusters-create.png 'Create new Databricks cluster')

4. On the Create Cluster page, enter **iot-cluster-SUFFIX** for the cluster name, leave the remaining values to their defaults, and select **Create Cluster**.

   ![On the Create Cluster page, "iot-cluster-SUFFIX" is entered into the Cluster Name field.](media/azure-databricks-create-cluster.png 'Create Azure Databricks cluster')

5. After a few minutes, your cluster will display as running.

   ![The iot-cluster-SUFFIX cluster is displayed under Interactive Clusters, and ](media/azure-databricks-interactive-clusters.png 'Databricks Interactive clusters')

### Task 5: Provision Power BI

If you do not already have a Power BI account:

1. Go to <https://powerbi.microsoft.com/features/>.

2. Scroll down until you see the **Try Power BI for free!** section of the page and select the **Try Free** button.

   ![Screenshot of the Try Power BI Pro for free page.](media/power-bi-try-free.png 'Try Power BI Pro for Free ')

3. On the page, enter your work email address (which should be the same account as the one you use for your Azure subscription), and select **Sign up**.

   ![The Get started page has a place to enter your work email address, and a sign up arrow.](media/power-bi-get-started.png 'Power BI Get started page')

4. Follow the on-screen prompts, and your Power BI environment should be ready within minutes.

   > **Note**: You can always return to your Power BI environment by navigating to <https://app.powerbi.com/>.

### Task 6: Connect to your Lab VM

In this task, you will create an RDP connection to your lab virtual machine (VM).

1. In the [Azure portal](https://portal.azure.com), select Resource groups in the Azure navigation pane, enter your resource group name (hands-on-lab-SUFFIX) into the filter box, and select it from the list.

   ![Resource groups is selected in the Azure navigation pane, "hands" is entered into the filter box, and the "hands-on-lab-SUFFIX" resource group is highlighted.](./media/resource-groups.png 'Resource groups list')

2. In the list of resources for your resource group, select the LabVM virtual machine.

   ![The list of resources in the hands-on-lab-SUFFIX resource group are displayed, and LabVM is highlighted.](./media/resource-group-resources-labvm.png 'LabVM in resource group list')

3. On your Lab VM blade, select Connect from the top menu.

   ![The LabVM blade is displayed, with the Connect button highlighted in the top menu.](./media/connect-labvm.png 'Connect to LabVM')

4. Select **Download RDP file**, then open the downloaded RDP file.

   ![The Connect to virtual machine blade is displayed, and the Download RDP file button is highlighted.](./media/connect-to-virtual-machine.png 'Connect to virtual machine')

5. Select **Connect** on the Remote Desktop Connection dialog.

   ![In the Remote Desktop Connection Dialog Box, the Connect button is highlighted.](./media/remote-desktop-connection.png 'Remote Desktop Connection dialog')

6. Enter the following credentials when prompted:

   - **User name**: demouser

   - **Password**: Password.1!!

7. Select **Yes** to connect, if prompted that the identity of the remote computer cannot be verified.

   ![In the Remote Desktop Connection dialog box, a warning states that the identity of the remote computer cannot be verified, and asks if you want to continue anyway. At the bottom, the Yes button is circled.](./media/remote-desktop-connection-identity-verification.png 'Remote Desktop Connection dialog')

8. Once logged in, launch the **Server Manager**. This should start automatically, but you can access it via the Start menu if it does not start.

   ![The Server Manager tile is circled in the Start Menu.](./media/start-menu-server-manager.png 'Server Manager tile in the Start menu')

9. Select **Local Server**, then select **On** next to **IE Enhanced Security Configuration**. **Note:** If the link says "Off", skip ahead to the next task.

   ![Screenshot of the Server Manager. In the left pane, Local Server is selected. In the right, Properties (For LabVM) pane, the IE Enhanced Security Configuration, which is set to On, is highlighted.](./media/windows-server-manager-ie-enhanced-security-configuration.png 'Server Manager')

10. In the Internet Explorer Enhanced Security Configuration dialog, select **Off under Administrators**, then select **OK**.

    ![Screenshot of the Internet Explorer Enhanced Security Configuration dialog box, with Administrators set to Off.](./media/internet-explorer-enhanced-security-configuration-dialog.png 'Internet Explorer Enhanced Security Configuration dialog box')

11. Close the Server Manager.

### Task 7: Download Smart Meter Simulator project

Fabrikam has provided a Smart Meter Simulator that they use to simulate device registration, as well as the generation and transmission of telemetry data. They have asked you to use this as the starting point for integrating their smart meters with Azure.

1. From your LabVM, download the starter project by downloading a .zip copy of the Intelligent vending machines GitHub repo.

2. In a web browser, navigate to the Internet of Things MCW repo: <https://github.com/Microsoft/MCW-Internet-of-Things>.

3. On the repo page, select **Clone or download**, then select **Download ZIP**.

   ![Download .zip containing the Intelligent vending machines repository](media/git-hub-download-repo.png 'Download ZIP')

   > **Note:** If Internet Explorer displays the following message, complete the sub-tasks below: "Your current security settings do not allow this file to be downloaded".

   - Select the Gear icon in the upper right-hand corner of Internet Explorer, then select **Internet options** from the menu.

   ![Internet Explorer internet options](media/internet-options.png 'Internet options')

   - In the Internet Options window, select the **Security** tab. Select the **Trusted sites** zone, then click the **Sites** button to edit the list of trusted sites.

   ![Internet Explorer options: Security tab](media/internet-explorer-security-tab.png 'Security tab')

   - `https://github.com` should automatically be added to the textbox. Click the **Add** button next to the URL to add it to the list of Websites below. Click **Close** when done.

   - Finally, click **OK** to apply the changes and close internet options.

4. Unzip the contents to the folder **C:\SmartMeter**.

5. Navigate to the `SmartMeterSimulator.sln` file within the `Hands-on lab\lab-files\starter-project` folder and open it with Visual Studio 2017.

6. Sign in to Visual Studio or create an account, if prompted.

7. If the Security Warning for SmartMeterSimulator window appears, uncheck _Ask me for every project in this solution_, and select **OK**.

   ![The SmartMeterSimulator Security Warning window has the option to "Ask me for every project in this solution" circled.](./media/visual-studio-security-warning.png 'SmartMeterSimulator Security Warning')

> **Note**: If you attempt to build the solution at this point, you will see many build errors. This is intentional. You will correct these in the exercises that follow.

You should follow all steps provided _before_ performing the Hands-on lab.
