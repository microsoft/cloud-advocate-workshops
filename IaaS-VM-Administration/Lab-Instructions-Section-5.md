# Optimizing Virtual Machine Workloads on Azure.

## Section Five: IaaS VM BCDR

This section requires that you have completed Section Zero and Exercise 2.3

### Exercise 5.1. Perform Virtual Machine Backup

As you have enabled automanage in Exercise 2.3, the Windwos Server IaaS VM is already configured for backups. Whilst Automanage has been configured, it is unlikely that a backup has been taken as you have only enabled this service relatively recently. You can perform manual backups in addition to the backups managed through automanage at any time. You perform a manual backup by performing the following steps:

5.1	You can see a quick audio free video showing the lab steps here: [Exercise 5.1 Demo Video](https://youtu.be/SZjoxpD9O8s)

1. In the Search bar of the Azure Portal, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-WS2022**.
2. On the SYD-WS2022 page Search bar at the top of the list of items, type **Backup**.
3. On the SYD-WS2022 page, note that backup is already configured by automanage but that the initial backup is pending and hasn’t been completed. Choose **Backup Now**.
4. On the **Backup Now** page, accept the default backup retention option and choose **OK**.

### Exercise 5.2: Configure Azure Site Recovery

Azure Site Recovery allows you to failover a virtual machine from one Azure region to another. In this exercise you'll configure replication for an encrypted virtual machine from one region to another.

5.2 You can see a quick audio free video showing the lab steps here: [Exercise 2.1 Demo Video](https://youtu.be/KcrxNGg7nAQ)

1. In the Search bar of the Azure Portal, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-WS2022**.
2. On the SYD-WS2022 page Search bar at the top of the list of items, type **Disaster Recovery**. Select **Disaster Recovery**.
3. On the SYD-WS2022 Disaster Recovery Page, accept the default Target Region (which will be different from the current region the VM is located in) and choose **Next: Advanced Settings**.
4. On the Advanced Settings page, review the automatically assigned resource group, virtual network, and key vault names that will be created and choose Next: **Review + Start Replication**.
5.  On the **Review + Start Replication** page, choose **Start Replication**. 

**At this point you will notice an error about incorrect permissions being configured for the existing and recovery site key vault. To enable Replication you will need to configure these permissions on both the source and destination key vaults. This is not necessary except in the case of BitLocker and DM-Crypt encrypted IaaS VMs.**

6.  Open the Azure Portal in a new browser tab and in the Search bar type **Resource Groups**. Select **Resource Groups**.
7.  In the list of resource groups, select InfraResourceGroup.
8.  In the **InfraResourceGroup** resource group, select the **IaaSVMKVXXXXX** key vault.
9.  On the **IaaSVMKVXXXX** key vault, select **Access Policies**.
10. On the **Access Policies** page, choose **Create**.
11. Add the following permissions and choose **Next**:
- Key permissions: **Key Management Operations: List, Create, and Get**
- Secret permissions: **Secret Management Operations: Get, List and Set**
12. On the **Principal** page, use **Search** to locate and choose the account you are signed in as (**User1-XXXXXXX@** - which is listed in the Resources tab) and choose **Select** and then choose **Next**.
13. On the **Review and Create** page, choose **Create**.
14. In the **Azure Portal Search Bar** type **Resource Groups**. Select **Resource Groups**.
15. In the list of resource groups, select **InfraResourceGroup-asr**.
16. In the **InfraResourceGroup-asr** resource group, select the **IaaSVMKVXXXXX-asr** key vault.
17. On the **IaaSVMKVXXXX-asr** key vault, select **Access Policies**.
18. On the **Access Policies** page, choose **Create**.
19. Add the following permissions and choose Next:
- Key permissions: **Key Management Operations: List, Create, and Get**
- Secret permissions: **Secret Management Operations: Get, List and Set**
20. On the Principal page, use **Search** to locate and choose the account you are signed in as (**User1-XXXXXXX@** - which is listed in the Resources tab) and choose **Select** and then choose **Next**.
21. On the **Review and Create** page, choose **Create**.
22. When the assignment is complete, switch back to the tab on which the replication error is present and choose **Start Replication**. This will take up to 15 minutes.

### Exercise 5.3. Perform VM Test Failover

1. Navigate back to the Disaster Recovery page for SYD-WS2022. When replication has successfully been enabled and the replication status is **Healthy**, click **Refresh**. An infrastructure view will display the virtual machine, Azure Site Recovery, and any managed disks.
2. Click **Refresh** until the Status field lists **Protected** and the **Test Failover** item becomes available.
3. On the SYD-WS2022 page, choose **Test Failover**.
4. On the Test Failover page, set the Azure Virtual Network to **InfraVNet-asr** and choose Test Failover.
5. Test Failover will take several minutes to complete.
6. When the Status shiftst to **Cleanup test failover pending**, click **Cleanup Test Failover pending**. This will open a log showing the different failover jobs and their completion status.
7. Return to the SYD-WS2022 Disaster Recovery Page and choose the **Cleanup test failover** item.
8. On the Notes page enter **“Failover Successful”** and then enable the Testing is complete checkbox and choose **OK**.
9. The test failover environment will be removed from the secondary datacenter.

### Congratulations you have finished this lab! 
### Please remember to complete an evaluation!