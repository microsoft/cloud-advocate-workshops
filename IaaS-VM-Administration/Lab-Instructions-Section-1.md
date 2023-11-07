# Optimizing Virtual Machine Workloads on Azure.


## Section One: Azure as IaaS VM Fabric

In this section you will use tools outside the VM to manage the configuration of the IaaS virtual machine. For example, you can use the IaaS Fabric to reassign the IP address of a VM, connect the VM to a different subnet, or even connect the VM to a different network.

### Exercise 1.1. Manage VM Network Configuration

In this section, you will perform manual configuration of an IaaS VM’s network settings. Unlike a normal virtual machine where you would edit IP address information from within the virtual machine using the operating system tools, the only way you can modify IP address settings for an Azure IaaS VM is by editing the properties of the network adapter from the Azure management plane. Some network address modifications require that the Azure IaaS VM be restarted. If the process appears to be frozen after 5 or more minutes, consider stopping and then restarting SYD-WS2022.

You can see a quick audio free video showing the lab steps here: [Exercise 1.1 Demo Video](https://youtu.be/vgOJi4T_11k)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-WS2022**.
2. In the SYD-WS2022 page, choose Network Settings under Networking.
3. On the Network Settings page, choose Network Interface. This will have a name similar to the VM and will be in the format syd-ws2022XYZ.
4. On the Network Interface page, choose IP configurations under Settings.
5. On the list of IP configurations, take note of the Private IP address. Select the IP configuration.
6. On the Edit IP configuration page, in the Private IP Address settings, set the Allocation to Static and set the new IP address to 10.0.0.40. Choose Save to save the new IP address.
7. Wait until the IP address is set (check the notification icon on the top bar of the portal)
8. Verify that on the IP settings page, the private IP address is set to 10.0.0.40.
9. On the Network Interface page, select DNS Servers under Settings.
10. On the DNS servers page, select Custom under DNS Servers.
11. In the DNS Server textbox, enter the IP address 1.1.1.1 and choose Save.
12. Once the new DNS server is added, in the breadcrumb menu at the top, select SYD-WS2022 to return to the VM properties page.
13. On the SYD-WS2022 properties page, search for Run Command in the search bar at the top of the VM properties page.
14. On the Run Command page, choose IPConfig.
15. On the Run Command Script page, choose Run.
16. On the Output page, verify which IP address configuration is displayed. Note that the IP address configuration includes the private IP address but not the public IP address visible in the portal.

### Exercise 1.2. Configure Network Security Groups

In this section of the exercise, you will manage the Network Security Group assigned to the IaaS VM network adapter. This network security group functions as a packet filter at the network adapter level. In this exercise you are going to create an inbound rule for port 80, so that you’ll be able to contact the web server that you’ll install on the Windows Server virtual machine.

You can see a quick audio free video showing the lab steps here: [Exercise 1.2 Demo Video](https://youtu.be/2v3SPyJg2UA)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-WS2022**.
2. In the **SYD-WS2022** page, choose **Network settings** under **Networking** (*Search for Networking if you cannot find it*).
3. On the **Networking** page, choose **Create Port Rule** (*or Add inbound port rule*) and then choose **Inbound port rule**.
4. On the **Add Inbound Security Rule** page, configure the following settings and choose Add.

- Source: **Any**
- Source Port **Ranges: \***
- Destination: **Any**
- Service: **HTTP**
- Action: **Allow**
- Priority: **1010**
- Name: **AllowAnyHTTPInbound**

5. Return to the SYD-WS2022 VM page.
6. On the SYD-WS2022 properties page, under **Payload** (*or Operations depending on the Portal configuration*), choose **Run Command**. (Search for **Run Command** if it is not present in this location)
7. On the **Run Command** page, choose **RunPowerShellScript**
8. In the **Run Command Script** dialog, type the following and choose **Run**.

+++Install-windowsfeature -name Web-Server -includeallsubfeature+++

9.  Wait until you receive the status of **Success** in the Output page. An informational message will inform you that script execution is in progress. This may take up to 15 minutes.
10. On the SYD-WS2022 return to **Overview** at the top of the item list.
11. On the Overview page, under **Networking**, copy the **public IP address** into a new browser tab and press **Enter**. *Make a note of this public IP address using Notepad in the lab VM as you will use it later in the exercise*.
12. Verify that the public IP address displays the Windows Server Internet Information Services default page. Close this browser tab displaying the Internet Information Services page.
13. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-LINUX**.
14. In the SYD-LINUX page, choose **Network settings** under **Networking** (*Search for Networking if you cannot find it*).
15. On the **Networking** page, choose **Create Port Rule** (*or Add inbound port rule*) and then choose **Inbound port rule**.
16. On the **Add Inbound Security Rule** page, configure the following settings and choose Add.

- Source: **Any**
- Source Port Ranges: **\***
- Destination: **Any**
- Service: **HTTP**
- Action: **Allow**
- Priority: **1010**
- Name: **AllowAnyHTTPInbound**

17. Return to the SYD-LINUX VM page.
18. Under **Connect** (*or Settings depending on the portal configuration*) on the SYD-LINUX page, choose **Connect**.
19. On the Connect page, choose Select under **SSH using Azure CLI**.
20. On the SSL using Azure CLI dialog, review the settings and then enabled the I understand just-in-time policy message and then choose **Configure + connect**.
21. After a few moments the Cloud Shell windows opens. Review the message and type **yes** and press Enter. If you get a token error involving time, restart the browser and return to step 31.
22. Enter the following commands to update the VM.

+++sudo apt update && sudo apt upgrade -y+++

23. If you are prompted by a Newer Kernel Available message or needing to restart the service due to outdated libraries, press **Tab** to select **OK** and press **Enter**. After this has occurred (or if it does not occur), enter the following to install ngingx.

+++sudo apt install nginx -y+++

24. Press **Tab** and select **OK** and press **Enter** if prompted about a pending Kernel upgrade or if you need to restart a service due to outdated libraries.
25. Type **exit** twice and select **Quit** to close the cloud shell connection to SYD-LINUX.
26. On the SYD-LINUX tools menu select **Overview** at the top of the item list.
27. On the Overview page, under **Networking**, copy the **public IP address** into a new browser tab and press **Enter**. Make a note of this public IP address using Notepad in the lab VM as you will use it later in the exercise.
28. Verify that the public IP address displays the **Welcome to nginx** default page. Close this browser tab displaying the nginx page.

### Exercise 1.3. Add disks to an IaaS VMs

In this exercise, you will add a disk to an IaaS VM running Ubuntu, connect to the VM using SSH from CloudShell, format the newly added disk and then mount it.

You can see a quick audio free video showing the lab steps here: [Exercise 1.3 Demo Video](https://youtu.be/sBeJlHAeTyk)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. Under **Settings** choose **Disks**.
4. On the **Disks** page, select **Create and Attach a New Disk**.
5. On the **Disks** section, provide the following information and choose Apply.
- Lun: **2**
- Disk name: **DiskGamma**
- Storage Type: **Premium SSD (locally-redundant storage)**
- Size: **4 GiB**
- Encryption: **Platform-managed key**

6. Under **Connect** (*or Settings depending on the portal configuration*) on the SYD-LINUX page, choose **Connect**.
7. On the **Connect** page, choose **Select** under **SSH using Azure CLI**.
8. On the **SSL using Azure CLI** dialog, review the settings and then enable the **I understand just-in-time policy** message and then choose **Configure + connect**. You may not receive this message and the process may move automatically to the next step.
9. After a few moments the Cloud Shell windows opens. Review the message and type **yes** and press Enter.
10. When signed in through SSH to the Linux machine at the Cloud Shell prompt, type the following command to list all the storage attached to the Linux IaaS VM taking note of the item sdc.

+++lsblk+++

11. Format the disk with ext4 by running the following command

+++sudo mkfs.ext4 /dev/sdc+++

12. Create a folder to mount the disk

+++sudo mkdir /mnt/newdisk+++

13. Mount the disk in the new folder

+++sudo mount -t ext4 /dev/sdc /mnt/newdisk+++

14. Verify that the disk is mounted by running the following commands

+++lsblk+++

+++df+++

15. If prompted to restart services choose **OK**
16. Enter the command **exit** twice to sign out and exit the Cloud Shell SSH session and then choose **Quit** to close the Cloud Shell pane.
17. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-WS2022**.
18. In the **SYD-WS2022** menu, select **Disks** under **Settings**.
10. On the **Disks** page, select **Create** and attach a new disk.
20. Configure the new data disk with the following settings and choose Apply.:

- LUN: **0**
- Disk name: **ExtraData**
- Storage Type: **Premium SSD**
- Size: **4 GiB**
- Encryption **Platform Managed Key**
- Host Caching: **Read-only**

21.You will configure this disk using Windows Admin Center in Exercise 2.2.

### Exercise 1.4. Resize a VM

In this exercise you will resize a VM running Linux from the Azure Portal.

You can see a quick audio free video showing the lab steps here: [Exercise 1.4 Demo Video](https://youtu.be/tWNp0Mfb9MM)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**. In the list of virtual machines, select **SYD-LINUX**.
2. Under **Settings** choose **Size**. You can also search for Size on the search bar at the top of the SYD-LINUX tools page.
3. On the list of VM sizes, search for and choose **D4s_v3** and then choose **Resize**.
4. After the operation completes (may take up to 15 minutes), select the **Overview** item and verify that the VM is now of the **Standard D4s_v3** size and has **4 vcpus** and **16 GiB** of memory.

===
Choose Next to go to Section 2