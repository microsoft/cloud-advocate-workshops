# Optimizing Virtual Machine Workloads on Azure.

## Section Four: IaaS VM Security

This section requires that you have completed section zero.

### Exercise 4.1. Configure BitLocker Encryption and DMCrypt

In this exercise you will configure BitLocker Encryption for the Windows Server IaaS VM and DMCrypt for the Linux VM. To complete this exercise, perform the following steps:

4.1 You can see a quick audio free video showing the lab steps here: [Exercise 4.1 Demo Video](https://youtu.be/F6lG_-qIZx4)

1. From the Azure Portal, open Cloud Shell
2. Run the following command to list the keyvault name

+++az keyvault list --output table+++

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

### Exercise 4.2. Configure Azure Software Update and Hotpatch

In this exercise you will configure Azure Software Update and Hotpatch

4.2	You can see a quick audio free video showing the lab steps here: [Exercise 4.2 Demo Video](https://youtu.be/jIcg8lhV8AQ)

1. In the Search bar of the Azure Portal, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. On the **SYD-LINUX** page, choose **Updates** under **Operations**.
4. On the **Updates** page, choose **Go to Updates using Azure Update Manager**.
5. On the SYD-LINUX updates page, choose **Enable Now** next to **Periodic Assessment**.
6. On the **Change Update Settings page**, set the **Periodic Assessment** drop down to **Enable** and choose **Save**.
7. In the Search bar of the Azure Portal, type **Virtual Machines** and then choose **Virtual Machines**.
8. In the list of virtual machines, select **SYD-WS2022**.
9. On the SYD-WS2022 page, choose **Updates** under **Operations**.
10. Chose **Go to Updates using Azure Update Manager**.
11. On the **Updates** page, choose **Check for Updates**.
12. When prompted whether you want to trigger an assessment, choose **OK**.
13. When the assessment completes, choose **Refresh**. If no updates are present, move on to Step 29.
14. Choose **One-Time Update**.
15. On the **Machines** page of the **Install one-time updates** wizard, select **SYD-WS2022** and choose **Next**.
16. On the **Updates** page, review the updates to install and choose **Next**.
17. On the **Reboot Option** page, select **Reboot if required** and choose **Next**.
18. On the **Review + Install** page, choose **Install**.
19. On the Updates page, choose **History** and then choose **Refresh**. This will display the progress of the update deployment. Wait until the status reaches **Succeeded**. You may have to wait several minutes and select **Refresh** multiple times.
20. Choose **Update settings**.
21. On the Change Update Settings page, configure the following settings and choose **Save**.
- Periodic Assessment: **Enable**
- Hotpatch: **Enable**

### Exercise 4.3. Configure Defender for Cloud Settings

In this exercise you will configure Defender for Cloud settings related to Azure IaaS VMs and view recommendations.

4.2	You can see a quick audio free video showing the lab steps here: [Exercise 4.3 Demo Video](https://youtu.be/EKMbq3Lwz0o)

1. In the Azure Portal Search bar, type **Defender for Cloud** and choose **Defender for Cloud** from the list of results.
2. In the Defender for Cloud portal, type *Environment settings* in the Search bar and then select *Environment Settings*.
3. On the Environment Settings page, choose Expand All and navigate down until you see the Azure subscription. It will have a name similar to *ignite-lodXXXXX*. Choose the Azure subscription to open it.
4. On the **Settings | Defender Plans** page, choose **Settings & Monitoring**.
5. On the **Settings & Monitoring** page, set **Vulnerability assessment for machines** to On.
6. On the **Extension Deployment Configuration** page, select **Microsoft Defender Vulnerability Management** and choose **Apply**.
7. On the **Extension Deployment Configuration** page, set **Guest Configuration Agent** to **On**. Choose **Continue** at the top of the page to apply settings and choose **Save** on the **Settings | Defender Plans** page.
8. In the Azure Portal Search bar, type **Virtual Machines** and choose **Virtual Machines** from the list of results.
9. On the list of virtual machines select **SYD-LINUX**.
10. On the **SYD-LINUX** page type Defender in the search bar and then choose **Defender for Cloud**.
11. On the list of recommendations for **SYD-LINUX** choose **Machines Should Have A Vulnerability Assessment Solution** if it is available. If this option is not available, continue on at Step 14.
12. On the **Machines Should Have A Vulnerability Assessment Solution** page, choose **Fix**.
13. On the **Choose A Vulnerability Assessment Solution** page, choose **Microsoft Defender Vulnerability Management** and choose **Proceed** and then choose **Fix 1 resource**.
14. In the Azure Portal Search bar, type **Virtual Machines** and choose **Virtual Machines** from the list of results.
15. In the list of virtual machines, choose **SYD-WS2022**.
16. On the **SYD-WS2022** page type Defender in the search bar and then choose **Defender for Cloud**.
17. On the **SYD-WS2022 | Defender for Cloud** page, review the recommendations and then choose **View Additional Recommendations in Defender for Cloud**.
18. Review the additional information in Defender for Cloud related to the security of SYD-WS2022.
===
Choose Next to go to Section 5