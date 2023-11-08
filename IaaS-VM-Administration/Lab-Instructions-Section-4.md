# Optimizing Virtual Machine Workloads on Azure.

## Section Four: IaaS VM Security

This section requires that you have completed section zero.

### Exercise 4.1. Configure BitLocker Encryption and DMCrypt

In this exercise you will configure BitLocker Encryption for the Windows Server IaaS VM and DMCrypt for the Linux VM. To complete this exercise, perform the following steps:

4.1 You can see a quick audio free video showing the lab steps here: [Exercise 4.1 Demo Video](https://youtu.be/F6lG_-qIZx4)

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

===
Choose Next to go to Section 5