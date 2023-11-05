# Optimizing Virtual Machine Workloads on Azure.

## Section Three: Monitor IaaS VMs

In this section you will integrate Azure Monitor into the IaaS VMs SYD-LINUX and SYD-WS2022.

### Exercise 3.1. Configure VM Insights

1. In the Search bar of the Azure Portal, type **Monitor** and then choose **Monitor**.
2. Under **Insights** choose **Virtual Machines**.
3. On the **Get Started** page, choose **Configure Insights**.
4. On the **Overview** page, select the **Not Monitored** page and choose **Enable** next to **SYD-LINUX**
5. On the **Azure Monitor Insights Onboarding** page, choose **Enable**.
6. On the **Monitoring Configuration** page, ensure that **Azure Monitor Agent** is selected and then choose **Create New** under **Data Collection Rule**.
7. On the **Create New Rule** page, provide the following information and then choose Create:
- Data Collection Rule name: **SYDLINUXINSIGHT**
- Enable processes and dependencies map: **Yes**.
- Subscription: **Your subscription**.
- Log Analytics Workspace: **LogAnalytics1**
8. On the **Monitoring configuration** page choose **Configure**.
9. Close the **Azure Monitor Insights Onboarding** page.
10. On the **Overview** page, select the **Not Monitored** page and choose **Enable** next to **SYD-WS2022**
11. On the **Azure Monitor Insights Onboarding** page, choose **Enable**.
12. On the **Monitoring Configuration** page, ensure that **Azure Monitor Agent** is selected and then choose **Create New** under **Data Collection Rule**.
13. On the **Create New Rule** page, provide the following information and then choose **Create**:
- Data Collection Rule name: **SYDWIN2022INSIGHT**
- Enable processes and dependencies map: **Yes**.
- Subscription: **Your subscription**.
- Log Analytics Workspace: **LogAnalytics1**
14. On the Monitoring configuration page choose **Configure**.
15. And then choose **Enable**.
16. Close the **Monitoring Configuration** page.
17. Select the **Monitored** column and verify that **SYD-LINUX** and **SYD-WS2022** have **Monitoring Coverage** enabled
18. With **Virtual Machines** selected under **Insights**, choose the **Performance** tab and review the performance telemetry.

### Exercise 3.2. VM Metrics

In this exercise you’ll use Metrics to create a readout of available memory, CPU utilization and Network Out total for each IaaS VM.

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. Under **Monitoring** choose **Metrics**.
4. On the **Metric** drop down, select **Available Memory Bytes**.
5. Choose **Add Metric**. Select **Percentage CPU**.
6. Choose **Add Metric**. Select **Network Out Total**.
7. Select **Line Chart** and then choose **Grid**.
8. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
9. In the list of virtual machines, select **SYD-WS2022**.
10. Under **Monitoring** choose **Metrics**.
11. On the **Metric** drop down, select **Available Memory Bytes**.
12. Choose Add Metric. Select Percentage CPU.
13. Choose Add Metric. Select Network Out Total.
14. Select Line Chart and then choose Grid.

### Exercise 3.3. Configure Azure Monitor for VM Data Collection Endpoint

1. In the **Azure Portal Search Bar**, enter **Monitor** and select **Monitor** from the list of results.
2. In the **Monitor** page, under **Settings**, choose **Data Collection Endpoints**.
3. On the **Data Collection Endpoints** page, choose **Create**.
4. Provide the following information:
- Endpoint Name: **IaaSVMCollectionEndpoint**
- Subscription: **Your subscription**
- Resource Group: **InfraResourceGroup**
- Region: **Choose the same region as that which is used by your resource group**
5. Choose **Review + Create** and then choose **Create**. This will create the data collection endpoint that you will use for monitoring IaaS VMs.

### Exercise 3.4. Configure Azure Monitor for VM Data Collection Rule

In this exercise, you will create a Data Collection Rule to collect a specific set of events from the Windows Server event logs.

1. In the **Azure Portal Search Bar**, enter **Monitor** and select **Monitor** from the list of results.
2. In the **Monitor** page, under **Settings**, choose **Data Collection Rules**.
3. On the **Data Collection Rules** page, choose **Create**.
4. On the **Create Data Collection Rule** page, configure the following settings and choose Next.
- Rule name: **WinVMDCR**
- Subscription: **Your subscription**
- Resource Group: **InfraResourceGroup**
- Region: **The region in which your InfraResourceGroup resource group is located.**
- Platform type: **Windows**
- Data collection endpoint: **IaaSVMCollectionEndpoint**
5. On the **Resources** page, choose **Add Resources**.
6. On the Select a scope page, expand the **InfraResourceGroup** resource group and enable the **SYD-WS2022** checkbox. Choose **Apply**.
7. On the **Create Data Collection Rule** page, choose Next: Collect and deliver.
8. On the **Collect and deliver** page of the **Collect Data Collection Rule** wizard, choose **Add Data Source**.
9. On the **Add Data Source** page, select **Windows Event Logs** from the **Data Source Type** dropdown. Select the following checkboxes:
- Application: **Critical, Error, Warning**
- Security: **Audit Failure**
- System: **Critical, Error, Warning**
10. Choose **Next: Destination**.
11. On the **Destination** page, select the **Azure Monitor Logs** destination that is configured for your subscription and the **LogAnalytics1** account and choose **Add Data Source**.
12. Choose **Review + Create** and then choose **Create**.

### Exercise 3.5. Modify VM Data Collection Rule to include IIS Logs

In this exercise, you will modify a Data Collection Rule to collect a specific set of events from the IIS log.

1. In the **Azure Portal Search Bar**, enter **Monitor** and select **Monitor** from the list of results.
2. In the **Monitor** page, under **Settings**, choose **Data Collection Rules**.
3. Choose the **WinVMDRC** rule in **InfraResourceGroup** resource group.
4. Under **Configuration**, choose **Data Sources**.
5. On the **Data Sources** page, choose **Add**.
6. On the **Add Data Source** page, select **IIS Logs** as the **data source type**.
7. Choose Next: **Destination**.
8. On the **Destination** page, ensure that the destination of Azure Monitor Logs is the log analytics workspace **LogAnalytics1** is selected and then choose **Add Data Source**.

### Exercise 3.6. Configure Network Connection Monitor

1. In the **Azure Portal Search Bar**, enter **Network Watcher** and select **Network Watcher** from the list of results.
2. Under **Monitoring**, choose **Connection Monitor**.
3. On the **Connection Monitor** page, choose **Create**.
4. On the **Basics** page of the **Create Connection Monitor** wizard, provide the following information and choose **Next: Test Groups**:
- Connection Monitor Name: **IaaSVMTest**
- Subscription: **Your Subscription**
- Region: **Same region as resource group**
5. On the **Add Test Group** details page, enter the **Test Group Name** as **VMTestGroup** and choose **Add Sources**.
6. On the **Add Sources** page, expand the **InfraVNet** and **InfraSubnetNodes** and enable the **SYD-LINUX** checkbox. Choose **Add Endpoints**.
7. On the **Add test group** details page, choose **Add Test Configuration**. On the **Add Test Configuration** Page, provide the following details and accept the other defaults:
-  Test Configuration Name: **Default HTTP**
-  Protocol: **HTTP**
-  Destination Port: **80**
8. Choose **Add Test Configuration**.
9. On the **Add test group details** page, choose **Add destinations**.
10. On the **Add destinations** page, expand **InfraVNet** and then expand **InfraSubnet** and enable the **SYD-WS2022** checkbox. Choose **Add Endpoints**.
11. On the **Add test group** details page, choose **Add Test Group**.
12. On the **Test Groups** page, choose **Next: Workspaces**.
13. On the** Workspace** page, choose **Custom Workspace** and ensure that **LogAnalytics1** is selected. Choose **Review and Create** and then choose **Create**.
14. Doing this will create the connection test as well as install the Network Watcher Extension on the Linux IaaS VM.
15. In the **Azure Portal Search Bar**, enter **Network Watcher** and select **Network Watcher** from the list of results.
16. Under **Monitoring** select **Connection monitor**.
17. Select **IaaSVMTest** and then choose **View Logs**.
18. Review the results of the log, checking the **TestResult** field. This should indicate (assuming that everything is configured correctly) that the Linux VM can send traffic to port 80 on the Windows Server VM with IIS installed and Port 80 open on the NSG rule.

### Exercise 3.7. Configure Performance Alerts and Action Groups

In this exercise you will configure a performance alert and an action to occur based on CPU utilization on a Linux IaaS VM.

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. On the SYD-LINUX page, choose **Alerts** under **Monitoring**.
4. On the **Alerts** page, choose **Create** and then choose **Alert Rule**.
5. On the **Condition** page of the **Create an Alert Rule** wizard, set the **Signal name** to **Percentage CPU**. Note in the Preview page the default CPU utilization is under 10% on average.
6. Accept the default settings and choose **Next: Actions**.
7. On the **Actions** page, choose **Create Action Group**.
8. On the **Create Action Groups** page, configure the following and then select **Next: Notifications**.
-  Subscription: **Your subscription**
-  Resource Group: **InfraResourceGroup**
-  Region: **Global**
-  Action Group Name: **NotifyCPU**
-  Display Name: **NotifyCPU**
9.  On the **Notifications** page, On the **notification type** drop down menu select **Email Azure Resource Manager Role** and set the name as **Email Admin**. Choose the pencil icon under **Selected**. On the **Email Azure Resource Manager Role** page, use the drop down and then select **Owner**. Choose **OK**. Choose **Next: Details**
10. On the **Details** page, enter the alert rule name **HighCPU**. Choose **Review + Create** and then choose **Create**.

===
Choose Next to go to Section 4