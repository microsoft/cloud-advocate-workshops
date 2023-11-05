# Optimizing Virtual Machine Workloads on Azure.

In this workshop you’re going to learn about Azure’s IaaS VM supporting technologies so that you can get the most out of your migrated and new IaaS VM workloads. You’ll learn how to apply each technology to your IaaS VMs through practical hands on steps, when and why you should implement these technologies, and tips and tricks on how you can get the most out of running migrated and new IaaS VM workloads in Azure.

## Section Zero: Lab Deployment

Lab Deployment

### Exercise 0.1. Verify Lab Deployment

To verify that the lab resources have deployed completely, perform the following steps:

1. When logged on to the VM you will use to connect to the lab instance, open Edge and log on to the Azure portal at portal.azure.com and using the username and passwords present on the green lab sidebar. You can click on the **[T]** icon to have sidebar text automatically typed into the VM window. (If the VM doesn’t have network connectivity, just reboot from the Start menu whilst connected through the remote web connection).
2. In the top bar next to the text entry area, select the Cloud Shell icon.
3. When prompted by the Welcome to Azure Cloud Shell page, choose Bash.
4. Accept the default information about storage and choose **Create Storage** (this is necessary to use Cloud Shell in later exercises).
5. Enter the following command to view the list of resources in the resource group:

+++az resource list --resource-group InfraResourceGroup --output table+++

6.  Type the following command to exit Cloud Shell

+++Exit+++

7. Choose **Quit** to close the Cloud Shell pane.
8. In the Search bar of the Azure Portal, type **Resource Groups** and select **Resource Groups**.
9. In the list of resource groups, Select the **InfraResourceGroup** resource group. Take a note of the Azure Region that your resource group is configured to use. You will deploy all resources in the lab to this region and will have to enter this region when using a variety of wizards. If you restart the lab from the beginning, you may be assigned to a new region.
10. The lab will be deployed when you see the following items present:
-  A virtual Machine with the name **SYD-LINUX**
-  A virtual machine with the name **SYD-WS2022**
-  Key Vault with the name **IaaSVMKVxxxxx** *Where xxxxx is a number to provide a unique name*.
11. You may need to wait several minutes and press refresh several times in the Azure portal.
12. If the VM resources do not become available after 15 minutes, cancel and restart the lab.

### Exercise 0.2. Register Microsoft Insights resource provider.

The Microsoft Insights resource provider is required to use the VM Insights functionality that we will enable later in this lab. This exercise is a pre-requisite for Section Three.

1. In the **Azure Portal Search Bar**, type Subscriptions.
2. Select the listed subscription.
3. In the Menu, under Settings, choose Resource Providers
4. Search for Microsoft Insights in the list of providers and then select microsoft.insights and choose Register.

### Exercise 0.3. Enable Defender for Cloud Licenses

In this exercise you’ll enable Defender for Cloud licenses to allow for functionality that will be used throughout the rest of the lab, such as Just In Time Access used when configuring Windows Admin Center. This exercise is a prerequisite for exercise 2.2 and all exercises in section four. You may need to refresh Defender for Cloud as it can take some time to make licenses available.

1. In the Azure Portal, type Defender for Cloud in the Search Bar and then select Defender for Cloud.
2. In the Defender for Cloud portal, choose Environment Settings.
3. On the Environment Settings page, choose Expand All.
4. At the bottom of expanded the list, choose the subscription which will have a name like build23-lodXXXXX. If this is listed as Unregistered, click Refresh and then click on the build23-lodXXXXX item.
5. On the Defender Plans page, next to Defender CSPM set the switch to On.
6. Next to Servers under Cloud Workload Protection (CWP) set the slider to On.
7. Next to Storage under Cloud Workload Protection (CWP) set the slider to On.
8. Next to Key Vault under Cloud Workload Protection (CWP) set the slider to On.
9. Next to Resource Manager under Cloud Workload Protection (CWP) set the slider to On.
10. Select Save at the top under Defender Plans.

