
# Internet of Things setup

## Requirements

-   Microsoft Azure subscription must be pay-as-you-go or MSDN.

    -   Trial subscriptions will not work.

-   A virtual machine configured with:

    -   Visual Studio Community 2017 15.6 or later

    -   Azure SDK 2.9 or later (Included with Visual Studio 2017)

    -   [R Tools for Visual Studio](https://aka.ms/rtvs-current) 0.3.2 or later

    -   [Power BI Desktop](https://powerbi.microsoft.com/desktop) (June 2016 build or later)

-   A running R Server on HD Insight Spark cluster 


## Before the hands-on lab

Duration: 45 minutes

In this exercise, you will set up your environment for use in the rest of the hands-on lab. You should follow all the steps provided in the Before the hands-on lab section to prepare your environment *before* attending the hands-on lab.

### Task 1: Provision Power BI

If you do not already have a Power BI account:

1.  Go to <https://powerbi.microsoft.com/features/>.

2.  Scroll down until you see the **Try Power BI for free!** section of the page and click the **Try Free\>** button.
    
    ![Screenshot of the Try Power BI Pro for free page.](images/Setup/image2.png "Try Power BI Pro for Free ")

3.  On the page, enter your work email address (which should be the same account as the one you use for your Azure subscription), and select **Sign up**.
    
    ![The Get started page has a place to enter your work email address, and a sign up arrow.](images/Setup/image3.png "Get started page")

4.  Follow the on-screen prompts, and your Power BI environment should be ready within minutes. You can always return to it via <https://app.powerbi.com/>.

### Task 2: Provision an HDInsight with Spark Cluster

Using the Azure Portal, provision a new HDInsight cluster.

1.  Open a browser, and go to the Azure portal (<https://portal.azure.com>).

2.  Select **+New**, select **Data + Analytics**, then select **HDInsight**.
    
    ![The Azure Portal has the New button selected in the left pane. In the New pane, under Azure Marketplace, Data + Analytics is selected, and under Featured, HDInsights is selected.](images/Setup/image4.png "Azure Portal, New pane")

3.  On the HDInsight blade, select **Custom (size, settings, apps)**.
    
    ![The HDInsight blade has the Custom (sie, settings, apps) selected.](images/Setup/image5.png "HDInsight blade")

4.  On the Basics blade, enter the following settings:

    -   Cluster name: Enter a unique name (verified by the green checkmark).

    -   Subscription: Select the Azure subscription into which you want to deploy the cluster.

    -   Custer type: Select ***Configure required settings***.

        ![Configure required settings link](images/Setup/image6.png "Configure required settings link")

        i.  On the Cluster configuration blade, set the **Cluster type** to **Spark** and the **Version** to **Spark 2.1.0 (HDI 3.6)**. Note that the Operating System option for the Spark cluster is fixed to Linux.
            
        ![Cluster configuration dialog with the Cluster type and Version optioins highlighted. The Cluster type is Spark and the version is Spark 2.1.0 (HDI 3.6)](images/Setup/image7.png "Cluster configuration dialog")

        ii. Select **Select** to close the Cluster configuration blade.

    -   Cluster login username: Leave as **admin**.

    -   Cluster login password: Enter **Password.1!!** for the admin password.

    -   Secure Shell (SSH) username: Enter **sshuser**.

    -   Use same password as cluster login: Ensure the checkbox is **checked**.

    -   Resource group: Select the Create new radio button, and enter **iot-hol** for the resource group name.

    -   Location: Select the desired location from the dropdown list, and remember this, as the same location will be used for all other Azure resources.
        
        ![The Basics blade fields display the previously mentioned settings.](images/Setup/image8.png "Basics blade")

    -   Select **Next** to move on to the storage settings.

5.  On the Storage blade:

    -   Primary storage type: Leave set to **Azure Storage.**

    -   Selection Method: Leave set to **My subscriptions**.

    -   Select a Storage account: Select Create new, and enter a name for the storage account, such as iotholstorage.

    -   Default container: Enter **iotcontainer**.

    -   Additional storage accounts: Leave unconfigured.

    -   Data Lake Store access: Leave unconfigured.

    -   Metastore Settings: Leave blank. 

        ![The Storage blade fields display the previously mentioned settings.](images/Setup/image9.png "Storage blade")

    -   Select **Next**.

6.  Select **Next** on the Applications (optional) blade. No applications are being added.

7.  On the Cluster size blade:

    -   Number of worker nodes: Leave set to **4**.

    -   Select **Worker node size**, and select **D12 v2**, then select **Select**.
        
        ![The Cluster size blade, worker node size section has the D12 V2 Standard option circled.](images/Setup/image10.png "Cluster size blade, worker node size section")

    -   Leave **Head node size**, set to the default, **D12 v2**.
        
        ![The Cluster size blade fields display the previously mentioned settings.](images/Setup/image11.png "Cluster Size Blade")

    -   Select **Next**.

8. Select **Next** on the Advanced settings blade to move to the Cluster summary blade.

9. Select **Create** on the Cluster summary blade to create the cluster.

10. It will take approximately 20 minutes to create your cluster. You can move on to the steps below while the cluster is provisioning.

### Task 3: Setup a lab virtual machine (VM)

1.  In the [Azure Portal](https://portal.azure.com/), select **+New**, then type "Visual Studio" into the search bar. Select **Visual Studio Community 2017 (latest release) on Windows Server 2016 (x64)** from the results**.
    
    ![In the Azure Portal, Everything pane, Visual studio is typed in the search field. Under Results, under Name, Visual Studio Community 2017 is circled.](images/Setup/image12.png "Azure Portal, Everything pane")

2.  On the blade that comes up, at the bottom, ensure the deployment model is set to **Resource Manager** and select **Create**.

    ![Resource Manager option](images/Setup/image13.png "Resource Manager option")

3.  Set the following configuration on the Basics tab.

-   Name: Enter **LabVM**.

-   VM disk type: Select **SSD**.

-   User name: Enter **demouser**

-   Password: Enter **Password.1!!**

-   Subscription: Select the same subscription you used to create your cluster in [Task 1](#task-1-provision-power-bi).

-   Resource Group: Select Use existing, and select the resource group you provisioned while creating your cluster in Task 1.

-   Location: Select the same region you used in Task 1 while creating your cluster.
    
    ![The Basics blade fields display with the previously mentioned settings.](images/Setup/image14.png "Basics blade")

4.  Select **OK** to move to the next step.

5.  On the Choose a size blade, ensure the Supported disk type is set to SSD, and select View all. This machine won't be doing much heavy lifting, so selecting **DS2\_V3** **Standard** is a good baseline option.
    
    ![The Choose a size blade, worker node size section has D2S\_V3 Standard circled. Supported disk type is set to SSD, and the View all button is circled.](images/Setup/image15.png "Choose a size blade, worker node size section")

6.  Select **Select** to move on to the Settings blade.

7.  Accept all the default values on the Settings blade, and Select **OK**.

8.  Select **Create** on the Create blade to provision the virtual machine.
    
    ![Screenshot of the Create blade, Summary section, showing that Validation passed, and detailing the offer details.](images/Setup/image16.png "Create blade, Summary section")

9.  It may take 10+ minutes for the virtual machine to complete provisioning.

### Task 4: Connect to the lab VM

1.  Connect to the Lab VM. (If you are already connected to your Lab VM, skip to Step 9.)

2.  From the left side menu in the Azure portal, click on **Resource groups**, then enter your resource group name into the filter box, and select it from the list.
    \
    ![In the Azure Portal, Resource groups is circled in the menu on the left. In the Resource groups pane, iot displays in the Subscriptions search field. Under Name, iot-hol is circled.](images/Setup/image17.png "Azure Portal, Resource groups pane")

3.  Next, select your lab virtual machine, **LabVM**, from the list.
    
    ![The LabVM option is selected.](images/Setup/image18.png "LabVM option")

4.  On your Lab VM blade, select **Connect** from the top menu.
    
    ![The Connect button is circled on the Lab VM blade menu bar.](images/Setup/image19.png "Lab VM blade menu bar")

5.  Download and open the RDP file.

6.  Select Connect on the Remote Desktop Connection dialog.
    
    ![The Connect button is circled on the Remote Desktop Connection dialog box, which asks if you still want to connect, even though the publisher of the remote connection can\'t be identified.](images/Setup/image20.png "Remote Desktop Connection dialog box")

7.  Enter the following credentials (or the non-default credentials if you changed them):

    -   User name: **demouser**

    -   Password: **Password.1!!

        ![Screenshot of the Windows Security, Enter your credentials window for demouser.](images/Setup/image21.png "Windows Security, Enter your credentials window")

8.  Select **Yes** to connect, if prompted that the identity of the remote computer cannot be verified.
    
    ![The Yes button is circled on the Remote Desktop Connection dialog box, which asks if you still want to connect, even though the identity of the remote connection can\'t be identified.](images/Setup/image22.png "Remote Desktop Connection dialog box")

9.  Once logged in, launch the Server Manager. This should start automatically, but you can access it via the Start menu if it does not start.

10. Select Local Server, then select On next to IE Enhanced Security Configuration.
    
    ![In Server Manager, in the left pane, Local Server is selected. In the right, Properties pane, a callout points to On, next to IE Enhanced Security Configuration.](images/Setup/image23.png "Server Manager")

11. In the Internet Explorer Enhanced Security Configuration dialog, select **Off** under Administrators, then select **OK**.
    
    ![On the Internet Explorer Enhanced Security Configuration dialog box, under Adminstrators, the Off radio button is selected and circled.](images/Setup/image24.png "Internet Explorer Enhanced Security Configuration dialog box")

12. Close the Server Manager.

### Task 5: Prepare an SSH client

In this task, you will download, install, and prepare the Git Bash SSH client that you will use to access your HDInsight cluster from your Lab VM.

1.  On your Lab VM, open a browser, and navigate to <https://git-scm.com/downloads> to download Git Bash.
    
    ![Screenshot of the Git Bash Downloads webpage.](images/Setup/image25.png "Download Git Bash")

2.  Select the download for your OS, and then select the Download 2.15.x for... button.

3.  Run the downloaded installer, select Next on each screen to accept the defaults.

4.  On the last screen, select Install to complete the installation.
    
    ![The Git Setup, Configuring extra options page has two checkboxes selected, for Enable file system caching, and Enable Git Credential Manager.](images/Setup/image26.png "Git Setup Wizard Configuring options page")

1. When the install is complete, you will be presented with the following screen:
    
    ![The Completing the Git Setup Wizard page has the check box selected to Launch Git Bash.](images/Setup/image27.png "Git Setup Wizard, Completing setup page")

6. Check **the Launch Git Bash** checkbox, and uncheck View Release Notes. Select **Finish**.

7. Leave the bash window open, as you will use it later in this lab.
