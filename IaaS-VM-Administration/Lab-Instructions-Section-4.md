# Optimizing Virtual Machine Workloads on Azure.

## Section Four: IaaS VM Security

This section requires that you have completed section zero.

### Exercise 4.1. VM Vulnerability Assessment and Just In Time access

[As this takes some time to become available, it may be that Exercise 4.1 needs to be skipped of VMs aren’t present in the Vulnerability Assessment sections and the JiT sections. To give this more time to become available, workshop attendees may choose to return to exercise 4.1 after they have completed section 5]

1. In the **Azure Portal search bar**, type **Defender for Cloud** and select **Defender for Cloud**.
2. On the **Defender for Cloud** menu, select **Workload Protections** under **Cloud Security**.
3. Under **Advanced Protection** choose **VM vulnerability assessment**.
4. On the **Machines should have vulnerability assessment solution** under **Affected resources**, select the checkboxes next to **syd-ws2022** and **syd-linux** and then choose **Fix**.
5. On the **Choose a vulnerability assessment solution** page, select **Microsoft Defender vulnerability Management** and choose **Proceed**.
6. On the **Fixing Resources** page, choose **Fix 2 resources**. Monitor the notification to verify remediation. This remediation will take several minutes to appear in the portal.
7. In the **Azure Portal** search bar, type **Defender for Cloud** and select **Defender for Cloud**.
8. On the **Defender for Cloud** menu, select **Workload Protections** under **Cloud Security**.
9. On the **Workload Protections** page, under **Advanced Protection**, choose **Just-In-Time VM Access**.
10. On the **Just-in-time VM access** page, you may see **SYD-WS2022** listed as you configured this setting indirectly when you configured Windows Admin Center. Select the **Not Configured** page and then select the checkbox next to **SYD-LINUX** (and SYD-WS2022 if it is not **present**). Chose **Enable JIT** on 1 (or 2) VMs.
11. On the **JIT VM Access Configuration** page, choose **Save**. Return to the **Configured** tab to verify that Just-in-time VM access is **enabled** for both **SYD-WS2022** and **SYD-LINUX**.
12. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
13. In the list of virtual machines, select **SYD-LINUX**.
14. Under **Connect** choose **Select** under **SSH using Azure CLI** and then enable the **I understand just-in-time policy** checkbox and select **Configure + Connect**.
15. Verify that you are able to make an SSH connection using Cloud Shell to SYD-LINUX. Once the connection is established, exit the connection by typing exit to close Cloud Shell. 
16. In the **Azure Portal search bar**, type **Defender for Cloud** and select **Defender for Cloud**.
17. On the **Defender for Cloud** menu, select **Workload Protections** under Cloud Security.
18. On the **Workload Protections** page, under **Advanced Protection**, choose **Just-In-Time VM Access**.
19. On the **Configured VMs** list, you will see the port 22 request for SYD-LINUX used by the account you used to connect to the Linux VM. Now that JiT is configured, all administrative connections will need to be requested and will be logged.
20. On the **Defender for Cloud** menu, select **Workload Protections** under **Cloud Security**.
21. On the **Workload Protections** page, under **Advanced Protection**, choose **Adaptive Network Hardening**.
22. On the **Adaptive Network Hardening** recommendations, choose the **Healthy Resources** tab. Both virtual machines will be listed here. Recommendations for these VMs based on analyzed traffic patterns.

### Exercise 4.2. Configure BitLocker Encryption and DMCrypt

In this exercise you will configure BitLocker Encryption for the Windows Server IaaS VM and DMCrypt for the Linux VM. To complete this exercise, perform the following steps:

1. From the Azure Portal, open Cloud Shell
2. Run the following command to list the keyvault name

+++Az keyvault list --output table+++

3. A name will be returned that will be in the format **IaaSVMKVxxxxxxxxx** where the string of xxxxxxx represents a number. Make note of this name (and perhaps post it into Notepad for later use) as you’ll need to substitute it into commands in step 5 and step 7. You might also select it and copy it so that you can paste it into those commands using SHIFT-INS in step 5 and step 7.
4. Type the following commands to verify that VM disk encryption is currently not enabled

+++az vm encryption show --name SYD-WS2022 -g InfraResourceGroup+++
+++az vm encryption show --name SYD-LINUX -g InfraResourceGroup+++

5. Enable BitLocker encryption on the Windows Server IaaS VM with the following command (changing the name of the Key Vault to match the one in your lab environment):

+++az vm encryption enable -g InfraResourceGroup --name SYD-WS2022 --disk-encryption-keyvault IaaSVMKVxxxxx+++

6. Verify that the VM is encrypted by running the following command (changing the name of the Key Vault to match the one in your lab environment):

+++az vm encryption show --name SYD-WS2022 -g InfraResourceGroup -o table+++

7. Enable DM-Crypt encryption on the Linux IaaS VM with the following command (changing the name of the Key Vault to match the one in your lab environment):

+++az vm encryption enable -g InfraResourceGroup --name SYD-LINUX --disk-encryption-keyvault IaaSVMKVxxxxx --volume-type ALL+++

8. Verify that the VM is encrypted by running the following command:

+++az vm encryption show --name SYD-LINUX -g InfraResourceGroup -o table+++

### Exercise 4.3. Configure Azure Software Update and Hotpatch

1. In the Search bar of the Azure Portal, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. On the **SYD-LINUX** page, choose **Updates** under **Operations**.
4. On the **Updates** page, choose **Go to Updates using Azure Update Manager**.
5. On the **Updates** page, choose **Check for Updates**.
6. When queried on triggering an assessment, choose **OK**.
7. After the assessment completes, click **Refresh** to review the total updates available. As you have run package updates on the Linux computer, you may not need to install any updates. If no updates are required, move on to Step 13. If updates are required, choose **One-time update**.
8. On the **Install one-time updates** page, select **SYD-LINUX** and choose **Next**.
9. On the **Updates to install** page, review the updates to be installed and choose **Next**.
10. On the **Reboot Options** page, select **Reboot if required** and choose **Next**.
11. On the **Review + Install** page, choose **Install**.
12. On the **Updates** page, choose **History** and then choose **Refresh**. This will display the progress of the update deployment. Wait until the status reaches **Succeeded**. You may have to wait several minutes and select **Refresh** multiple times.
13. On the SYD-LINUX updates page, choose **Enable Now** next to **Periodic Assessment**.
14. On the **Change Update Settings page**, set the **Periodic Assessment** drop down to **Enable** and choose **Save**.
15. In the Search bar of the Azure Portal, type **Virtual Machines** and then choose **Virtual Machines**.
16. In the list of virtual machines, select **SYD-WS2022**.
17. On the SYD-WS2022 page, choose **Updates** under **Operations**.
18. Chose **Go to Updates using Azure Update Manager**.
19. On the **Updates** page, choose **Check for Updates**.
20. When prompted whether you want to trigger an assessment, choose **OK**.
21. When the assessment completes, choose **Refresh**. If no updates are present, move on to Step 29.
22. Choose **One-Time Update**.
23. On the **Machines** page of the **Install one-time updates** wizard, select **SYD-WS2022** and choose **Next**.
24. On the **Updates** page, review the updates to install and choose **Next**.
25. On the **Reboot Option** page, select **Reboot if required** and choose **Next**.
26. On the **Review + Install** page, choose **Install**.
27. On the Updates page, choose **History** and then choose **Refresh**. This will display the progress of the update deployment. Wait until the status reaches **Succeeded**. You may have to wait several minutes and select **Refresh** multiple times.
28. Choose **Update settings**.
29. On the Change Update Settings page, configure the following settings and choose **Save**.
- Periodic Assessment: **Enable**
- Hotpatch: **Enable**
30. On the SYD-WS2022 **Updates** page, choose **Schedule Updates**.
31. On the Create a maintenance Configuration Page, configure the following settings and choose **Add a schedule**.
- Subscription: **Your subscription**
- Resource Group: **InfraResourceGroup**
- Configuration Name: **LabUpdateConfig**
- Region: Your Region (**determined in Exercise 0.1**)
- Maintenance Scope: **Guest**
- Reboot Setting **Reboot if Required**
32. On the **Add/Modify schedule** page, provide the following information and choose **Save**.
-  Start on **12/01/2023 12:00 AM**
-  Maintenance Windows **3 hours 55 minutes**
-  Repeats **1 Day**
33. On the **Create a maintenance Configuration** page, choose **Next: DynamicScopes**.
34. On the Dynamic Scopes tab, choose Add a dynamic scope.
35. On the **Add a dynamic scope** page, next to subscriptions select your subscription. Choose **Save**. When warned that not all Azure Machines shown in the preview are configured to schedule updates choose **Continue with supported machines only** and choose **Save**.
36. On the **dynamic scopes** page, choose **Next: Resources**.
37. On the **Resources** page, choose **Next: Updates**.
38. On the **Updates** page, review the update classifications and choose **Review + Create**.
39. On the **Review + Create** page, wait for validation to occur and then choose **Create**.

