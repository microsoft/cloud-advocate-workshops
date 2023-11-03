# Optimizing Virtual Machine Workloads on Azure.

You’ve taken that first step into the cloud and migrated on-premises servers into Azure. But migrating to Azure is only the first part of getting the most out of IaaS virtual machines deployed in the Azure cloud.

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

```Exit```

7. Choose **Quit** to close the Cloud Shell pane.
8. In the Search bar of the Azure Portal, type **Resource Groups** and select **Resource Groups**.
9. In the list of resource groups, Select the **InfraResourceGroup** resource group. Take a note of the Azure Region that your resource group is configured to use. You will deploy all resources in the lab to this region and will have to enter this region when using a variety of wizards. If you restart the lab from the beginning, you may be assigned to a new region.
10. The lab will be deployed when you see the following items present:
    -  A virtual Machine with the name SYD-LINUX
    -  A virtual machine with the name SYD-WS2022
    -  Key Vault with the name "IaaSVMKV".
11. You may need to wait several minutes and press refresh several times in the Azure portal.
12. If the VM resources do not become available after 15 minutes, cancel and restart the lab.

### Exercise 0.2. Register Microsoft Insights resource provider.

The Microsoft Insights resource provider is required to use the VM Insights functionality that we will enable later in this lab. This exercise is a pre-requisite for Section Three.

1. In the Azure Portal Search Bar, type Subscriptions.
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

===

## Section One: Azure as IaaS VM Fabric

In this section you will use tools outside the VM to manage the configuration of the IaaS virtual machine. For example, you can use the IaaS Fabric to reassign the IP address of a VM, connect the VM to a different subnet, or even connect the VM to a different network.

### Exercise 1.1. Manage VM Network Configuration

In this section, you will perform manual configuration of an IaaS VM’s network settings. Unlike a normal virtual machine where you would edit IP address information from within the virtual machine using the operating system tools, the only way you can modify IP address settings for an Azure IaaS VM is by editing the properties of the network adapter from the Azure management plane. Some network address modifications require that the Azure IaaS VM be restarted.

1. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-WS2022.
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

1. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-WS2022.
2. In the SYD-WS2022 page, choose Network settings under Networking (Search for Networking if you cannot find it).
3. On the Networking page, choose Create Port Rule (or Add inbound port rule) and then choose Inbound port rule.
4. On the Add Inbound Security Rule page, configure the following settings and choose Add.

- Source: Any
- Source Port Ranges: \*
- Destination: Any
- Service: HTTP
- Action: Allow
- Priority: 1010
- Name: AllowAnyHTTPInbound

5. Return to the SYD-WS2022 VM page.
6. On the SYD-WS2022 properties page, under Payload (or Operations depending on the Portal configuration), choose Run Command. (Search for **Run Command** if it is not present in this location)
7. On the Run Command page, choose RunPowerShellScript
8. In the Run Command Script dialog, type the following and choose Run.

```Install-windowsfeature -name Web-Server -includeallsubfeature```

9.  Wait until you receive the status of Success in the Output page. An informational message will inform you that script execution is in progress. This may take up to 15 minutes.
10. On the SYD-WS2022 return to Overview at the top of the item list.
11. On the Overview page, under Networking, copy the public IP address into a new browser tab and press Enter. Make a note of this public IP address using Notepad in the lab VM as you will use it later in the exercise.
12. Verify that the public IP address displays the Windows Server Internet Information Services default page. Close this browser tab displaying the Internet Information Services page.
13. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-LINUX.
14. In the SYD-LINUX page, choose Network settings under Networking (Search for Networking if you cannot find it).
15. On the Networking page, choose Create Port Rule (or Add inbound port rule) and then choose Inbound port rule.
16. On the Add Inbound Security Rule page, configure the following settings and choose Add.

- Source: Any
- Source Port Ranges: \*
- Destination: Any
- Service: HTTP
- Action: Allow
- Priority: 1010
- Name: AllowAnyHTTPInbound

17. Return to the SYD-LINUX VM page.
18. Under Connect (or Settings depending on the portal configuration) on the SYD-LINUX page, choose Connect.
19. On the Connect page, choose Select under **SSH using Azure CLI**.
20. On the SSL using Azure CLI dialog, review the settings and then enabled the I understand just-in-time policy message and then choose **Configure + connect**.
21. After a few moments the Cloud Shell windows opens. Review the message and type **yes** and press Enter. If you get a token error involving time, restart the browser and return to step 31.
22. Enter the following commands to update the VM.

``` Sudo apt update && sudo apt upgrade -y```

23. If you are prompted by a Newer Kernel Available message, press Tab to select OK and press Enter. After this has occurred (or if it does not occur), enter the following to install ngingx.

```Sudo apt install nginx -y```

24. Press Tab and select OK and press Enter if prompted about a pending Kernel upgrade.
25. Type exit twice and select Quit to close the cloud shell connection to SYD-LINUX.
26. On the SYD-LINUX tools menu select Overview at the top of the item list.
27. On the Overview page, under Networking, copy the public IP address into a new browser tab and press Enter. Make a note of this public IP address using Notepad in the lab VM as you will use it later in the exercise.
28. Verify that the public IP address displays the Welcome to nginx default page. Close this browser tab displaying the nginx page.

### Exercise 1.3. Add disks to an IaaS VMs

In this exercise, you will add a disk to an IaaS VM running Ubuntu, connect to the VM using SSH from CloudShell, format the newly added disk and then mount it.

1. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2. In the list of virtual machines, select SYD-LINUX.
3. Under Settings choose Disks.
4. On the Disks page, select Create and Attach a New Disk.
5. On the Disks section, provide the following information and choose Apply.

- Lun: 2
- Disk name: DiskGamma
- Storage Type: Premium SSD (locally-redundant storage)
- Size: 4 GiB
- Encryption: Platform-managed key

6. Under Connect (or Settings depending on the portal configuration) on the SYD-LINUX page, choose Connect.
7. On the Connect page, choose Select under **SSH using Azure CLI**.
8. On the SSL using Azure CLI dialog, review the settings and then enabled the I understand just-in-time policy message and then choose **Configure + connect**. You may not receive this message and the process may move automatically to the next step.
9. After a few moments the Cloud Shell windows opens. Review the message and type **yes** and press Enter.
10. When signed in through SSH to the Linux machine at the Cloud Shell prompt, type the following command to list all the storage attached to the Linux IaaS VM taking note of the item sdc.

```lsblk```

11. Format the disk with ext4 by running the following command

```Sudo mkfs.ext4 /dev/sdc```

12. Create a folder to mount the disk

```Sudo mkdir /mnt/newdisk```

13. Mount the disk in the new folder

```Sudo mount -t ext4 /dev/sdc /mnt/newdisk```

14. Verify that the disk is mounted by running the following commands

```Lsblk```

```df```

15. Enter the following commands to update the VM and install Python2

```Sudo apt update && sudo apt upgrade -y```

16. You may be prompted to restart services. If you get this message, press TAB and choose OK. Enter the following command to install Python.

```Sudo apt install python2 -y```    

17. If prompted to restart services choose OK.
18. Enter the command exit twice to sign out and exit the Cloud Shell SSH session and then choose Quit to close the Cloud Shell pane.
19. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-WS2022.
20. In the SYD-WS2022 menu, select Disks under Settings.
21. On the Disks page, select Create and attach a new disk.
22. Configure the new data disk with the following settings and choose Apply.:

- LUN: 0
- Disk name: ExtraData
- Storage Type: Premium SSD
- Size: 4 GiB
- Encryption Platform Managed Key
- Host Caching: Read-only

23. You will configure this disk using Windows Admin Center in Exercise 2.2.

### Exercise 1.4. Resize a VM

In this exercise you will resize a VM running Linux from the Azure Portal.

1. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-LINUX.
2. Under Settings choose Size. You can also search for Size on the search bar at the top of the SYD-LINUX tools page.
3. On the list of VM sizes, search for and choose D4s_v3 and then choose Resize.
4. After the operation completes (may take up to 15 minutes), select the Overview item and verify that the VM is now of the Standard D4s_v3 size and has 4 vcpus and 16 GiB of memory.

## Section Two: Manage IaaS VM Configuration

Section two only requires that you have performed section zero and the parts of exercise 1.3 where you added an extra disk to SYD-WS2022.

### Exercise 2.1. Enable Boot Diagnostics

In this exercise, you will enable boot diagnostics on Azure IaaS Virtual Machines.

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-WS2022.
3.  Under Monitoring, choose Diagnostic settings.
4.  On the Install Diagnostics Extension page, under Diagnostics Storage Account, choose the Storage Account named iaasbootdiagnosticsXXXX (where XXXX is a unique number) and choose Enable Guest-level Monitoring.
5.  Once the installation completes, under **Help**, choose **Boot Diagnostics**.
6.  On the Boot Diagnostics page, choose Settings.
7.  On the Boot Diagnostics Settings page, choose Enable with Custom Storage Account and on the Diagnostics Storage Account drop down select iaasbootdiagnosticsXXXX. Choose Apply.
8.  When the configuration completes, in the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
9.  In the list of virtual machines, select SYD-LINUX.
10. Under Help on the SYD-LINUX properties page, choose Boot Diagnostics.
11. On the Boot Diagnostics page, choose Settings.
12. On the Boot Diagnostics page, choose Enable with custom storage account and choose the Storage Account named iaasbootdiagnosticsXXXX and select Apply.
13. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
14. In the list of virtual machines, select SYD-WS2022.
15. Under Help choose Boot Diagnostics. You should now see an image of a locked Windows Server screen displaying the time and date. If a screen shot is not present, choose Refresh.
16. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
17. In the list of virtual machines, select SYD-LINUX.
18. Under Help choose Boot Diagnostics. You should not see an image of a Linux system waiting for sign on. If a screen shot is not present, choose Refresh.

### Exercise 2.2. Configure and administer Windows Server with Windows Admin Center

In this exercise, you will configure a Windows Server IaaS VM to be managed from the Azure portal using Windows Admin Center.

1. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2. In the list of virtual machines, select SYD-WS2022.
3. Under **Connect** choose **Windows Admin Center**.
4. On the Windows Admin Center page, review the inbound port and enable the **Open an outbound port for Windows Admin Center** to install and choose **Install**.
5. After installation completes, select Access control (IAM) and then choose Add role assignment.
6. Choose **Windows Admin Center Administrator Login** and choose Next.
7. Choose Select members and use Search to locate and choose the account you are signed in as (User1-XXXXXXX@ - which is listed in the Resources tab) and choose **Select**.
8. Choose **Review and Assign** and then choose **Review and Assign** again.
9. On the SYD-WS2022 page, choose Connect under Settings.
10. On the Connect page, under Windows Admin Center, choose Connect In Browser. On the Windows Admin Center pane, choose Configure. (If you encounter an error, choose the separate Windows Admin Center item in the VM menu pane and choose the Public IP address option. If that doesn’t work, you may also need to restart the IaaS VM)
11. In the Windows Admin Center Tools menu, choose Storage. You will see a set of virtual disks. One of these will not be initialized as you created and attach the disk in Exercise 1.3.
12. Select the not initialized and choose Initialize Disk. Initialize the disk with the GPT partition style and choose Submit.
13. Select the newly initialized disk and choose Create Volume.
14. On the Create Volume page, configure the following settings and choose Submit:
- Drive Letter: O
- Volume Label: Data Disk
- File System: ReFS
- Allocation unit size: 4096
- Use maximum size: Enable checkbox
15. On the Storage page, choose Volumes and review the volumes attached to SYD-WS2022.
16. In Windows Admin Center, choose Files & File Sharing, and then choose File Shares.
17. Choose File Server Settings.
18. Choose the following settings and choose Save.
- SMB signing: Required
- SMB 3 encryption: Required from all clients that support it
- Request SMB compression: Enabled
19. On the Windows Admin Center Tools menu, choose Certificates. Expand Overview and view the Expired certificates.
20. On the Windows Admin Center Tools menu, choose Firewall.
21. On the Firewall page, choose Incoming rules. Locate the World Wide Web Services (HTTP Traffic-In) rule and verify that it is set to Allowed. This rule was configured when you installed IIS.
22. On the Windows Admin Center Tools menu, select Roles & Features.
23. On the list of Roles and Features, expand File and Storage Services and then expand File and iSCSI Services.
24. Select Data Deduplication and then choose Install under Roles and Features near the top of the page.
25. Choose Yes on the Install Roles and Features page.
26. On the Tools menu, choose Remote Desktop.
27. On the Remote Desktop login credentials page, provide the following credentials:
- Username: .\\Prime
- Password: P@ssw0rdP@ssw0rd
- Automatically connect with the certificate presented by this machine: Enabled
28. Choose Confirm when prompted and then choose Connect.
29. You will be connected to a Remote Desktop session over the browser and the Windows Server Desktop Experience will be displayed. Click Cancel to Dismiss the Shutdown Event Tracker dialog box (this will be present due to reboots from earlier exercises). If you have performed this part of the lab after section 1, Internet Information Services will be shown as a node in Server Manager.
30. Through the web based Remote Desktop session, right click on the Start hint and choose Windows PowerShell (Admin)
31. At the PowerShell prompt type the following command and press Enter to view the computer name:
```Hostname```

To view the configured IP address settings, type the following command and press Enter:

Get-NetIPAddress

Verify that Deduplication has installed by running the following command:

Get-WindowsFeature FS-Data-Deduplication

Choose Disconnect under Remote Desktop to end the Remote Desktop Session.

On the Windows Admin Center Tools menu, choose Processes. This will allow you to see the processes that are currently running on SYD-WS2022. Locate the HotPatch process. This process is new to the Azure Edition of Windows Server 2022. You will learn more about HotPatch later in this lab.

On the Windows Admin Center Tools menu, choose Registry. This tool allows you to view and modify the registry on the Windows Server computer remotely without signing on directly to the Windows Server computer. On the Registry page, expand the following hive HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion and locate the ProductName key. This informs you of the version of Windows Server.

On the Windows Admin Center Tools menu, select Packet Monitoring. On the Consent to installing additional software page, enable both the I Agree To The License Terms And Consent To Installing The Software Above and Automatically Open Packet Monitoring Logs After Logging Has Been Stopped checkboxes and choose Consent To Installation.

On the Packet Monitoring page, choose New Capture.

On the Filter the captured packets by PacketMon page choose Add. On the Capture Filters enter IP Address 1 as 10.0.0.40 (the IP address you configured in Exercise 1.1) and Port 1 as 80 and then choose Start. Open a new browser tab and navigate to the public IP address of the IIS web server hosted on the Windows Server IaaS VM. Switch back to the original browser tab and choose Stop under Packet Monitoring. The results of the packet capture will be displayed, showing traffic connecting to the VM from a remote IP address and interacting with the IIS web server.

On the Windows Admin Center Tools menu, choose Services. On the Services page, choose Status at the top of the Status column to sort services into Running and Stopped. Select the Spooler service and then choose Restart. On the Restart Service pop up, choose Yes.

### Exercise 2.3. Configure Automanage for Azure VM

In this exercise you will configure Automanage for SYD-WS2022. Automanage allows you to automatically configure a number of Azure IaaS services that support Azure VMs without having to configure each on and individual basis.

Note: Automanage for current Ubuntu 22.04 image not supported (but is for prior images).

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-WS2022.
3.  On the SYD-WS2022 page, choose Automanage under Operations.
4.  On the Automanage page, choose Custom Profile.
5.  On the Custom Profile drop down select Create New.
6.  On the Create A Custom Profile Page, enter or confirm the following settings and choose Create:
    -   Name: AutomanageLab
    -   Subscription: Your Subscription
    -   Resource Group: InfraResourceGroup
    -   Region: Your region
    -   Enable Backup: Ensure checkbox is checked.
    -   Frequency: Daily
    -   At: 1:00 AM
    -   Timezone UTC
    -   Retain Instant Recovery Snapshots for 2
    -   Retention of daily backup point: 180 days
    -   Enable Microsoft Antimalware: Ensure checkbox is checked.
    -   Enable Real-time protection: Ensure checkbox is checked.
    -   Enable run a schedule scan: Ensure checkbox is checked.
    -   Scan type: Full
    -   Scan Day: Sunday
    -   Scan time: 120
    -   Enable Machine Insights Monitoring: Ensure checkbox is checked.
    -   Enable security baseline: Uncheck checkbox. (You will enable this manually later)
    -   Enable update management: Uncheck checkbox (You will enable this manually later)
    -   Enable change tracking and inventory: Uncheck checkbox (You will enable this manually later)
    -   Enable Azure Boot Diagnostics: Uncheck checkbox (you already performed this step in Exercise 2.1)
    -   Enable Windows Admin Center: Uncheck checkbox (you already performed this step in Exercise 2.2)
7.  With the Custom Profile set to AutomanageLab, choose Enable.

### Exercise 2.4. Configure Azure Policy for IaaS VM

In this exercise you will use Azure Policy to evaluate several elements of the configuration of IaaS VMs. Policy can be a useful tool to allow you to determine if your IaaS VMs are configured correctly or require manual or automatic remediation.

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-LINUX.
3.  On the SYD-LINUX page, select Policies under Operations.
4.  On the SYD-LINUX Policies page, choose Assign Policy.
5.  On the Assign Policy page, select the ellipsis (…) next to Policy definition.
6.  On the list of Available Definitions, type SSH in the Search textbox and select the Authentication to Linux machines should require SSH keys checkbox and choose Add.
7.  On the Assign Policy page, choose Review and Create and then choose Create.
8.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
9.  In the list of virtual machines, select SYD-WS2022.
10. On the SYD-WS2022 Policies page, search for Policies and choose Assign Policy.
11. On the Assign Policy page, select tie ellipsis (…) next to Policy definition.
12. On the Available Definitions page, in the search bar type Certificates. Select the Certificates using RSA cryptography should have the specified minimum key size policy and choose Add. Choose Next on the Basics page of the Assign Policy wizard.
13. On the Advanced page of the Assign Policy wizard, choose Next.
14. On the Parameters page, use the drop down menu and select 2048. Choose Review and Create and then choose Create.
15. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
16. In the list of virtual machines, select SYD-LINUX. On the SYD-LINUX Policies page, choose Refresh until the Compliance State changes to Non-compliant. This is because SYD-LINUX allows sign in via username and password to simplify the operation of this lab exercise. You can come back to this step later after you have completed exercise 2.6 if you don’t want to wait for the compliance state to change.
17. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
18. In the list of virtual machines, select SYD-WS2022. On the SYD-WS2022 Policies page, choose Refresh until the Compliance State of the Certificates using RSA policy changes to Compliant. This may take several minutes. Note that the Authentication to Linux Machines policy will be listed as Compliant because SYD-WS2022 is not running the Linux operating system. You can come back to this step later after you have completed exercise 2.6 if you don’t want to wait for the compliance state to change.

### Exercise 2.5. Connect via Serial Connection to IaaS VM

The serial console connection allows you to make a special administrative connection to an IaaS VM when there may be some sort of fault with the network configuration. Serial Console allows you to perform actions against the VM from a command line environment even if the network connection doesn’t work. You can use this for troubleshooting or diagnosis purposes. You can make serial console connections to Linux IaaS VMs and Windows Server IaaS VMs.

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-LINUX.
3.  Under Help choose Serial Console.
4.  When prompted, login with the following credentials:

Login: prime

Password: P@ssw0rdP@ssw0rd

1.  Type the following command to see information about your login

w \$(whoami)

1.  Type the following to verify that SYD-LINUX has network connectivity.

    ping [www.azure.com](http://www.azure.com)

2.  Use CTRL-C to stop the output of the ping command.
3.  Type exit to sign off from the serial console connection to the Linux IaaS VM.
4.  On the SYD-LINUX page Under Help choose Boot Diagnostics and then choose Serial log.
5.  You should see a log record of the commands you issued when connected via the Serial connection.
6.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
7.  In the list of virtual machines, select SYD-WS2022.
8.  Under Help choose Serial Console.
9.  Once the console connection is established, At the SAC\> prompt, type CMD
10. The message will return that a new channel named Cmd001 has ben created.
11. At the SAC\> prompt, type

    ch -sn Cmd0001

12. Press Enter to view the channel. You will be prompted to log on to the computer. Provide the following credentials:

    Username: prime

    Domain: \<Press Enter to provide no domain information\>

    Password: P@ssw0rdP@ssw0rd

13. You will be presented with the Windows Server command prompt. Type the following command to verify network connectivity:

    Ping [www.azure.com](http://www.azure.com)

14. Type sconfig to load the Windows Server command line configuration utility
15. On the sconfig menu, type 8 to view the network settings.
16. Type the number associated with the network adapter (it will be 1 or 2).
17. Press Enter to return to the main menu and then 12 to log off the user.
18. When prompted if you want to log off, choose Y.
19. Type exit to return to the SAC environment.
20. On the SYD-WS2022 page Under Help choose Boot Diagnostics and then choose Serial log.
21. You should see a log record of the commands you issued when connected via the Serial connection.

## Section Three: Monitor IaaS VMs

In this section you will integrate Azure Monitor into the IaaS VMs SYD-LINUX and SYD-WS2022.

### Exercise 3.1. Configure VM Insights

1.  In the Search bar of the Azure Portal, type Monitor and then choose Monitor.
2.  Under Insights choose Virtual Machines.
3.  On the Get Started page, choose Configure Insights.
4.  On the Overview page, select the Not Monitored page and choose Enable next to SYD-LINUX
5.  On the Azure Monitor Insights Onboarding page, choose Enable.
6.  On the Monitoring Configuration page, ensure that Azure Monitor Agent is selected and then choose Create New under Data Collection Rule.
7.  On the Create New Rule page, provide the following information and then choose Create:
    -   Data Collection Rule name SYDLINUXINSIGHT
    -   Enable processes and dependencies map: Yes.
    -   Subscription: Your subscription.
    -   Log Analytics Workspace: LogAnalytics1
8.  On the Monitoring configuration page choose Configure.
9.  Close the Azure Monitor Insights Onboarding page.
10. On the Overview page, select the Not Monitored page and choose Enable next to SYD-WS2022
11. On the Azure Monitor Insights Onboarding page, choose Enable.
12. On the Monitoring Configuration page, ensure that Azure Monitor Agent is selected and then choose Create New under Data Collection Rule.
13. On the Create New Rule page, provide the following information and then choose Create:
    1.  Data Collection Rule name SYDWIN2022INSIGHT
    2.  Enable processes and dependencies map: Yes.
    3.  Subscription: Your subscription.
    4.  Log Analytics Workspace: LogAnalytics1
14. On the Monitoring configuration page choose Configure.
15. And then choose Enable.
16. Close the Monitoring Configuration page.
17. Select the Monitored column and verify that SYD-LINUX and SYD-WS2022 have Monitoring Coverage enabled
18. With Virtual Machines selected under Insights, choose the Performance tab and review the performance telemetry.

### Exercise 3.2. VM Metrics

In this exercise you’ll use Metrics to create a readout of available memory, CPU utilization and Network Out total for each IaaS VM.

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-LINUX.
3.  Under Monitoring choose Metrics.
4.  On the Metric drop down, select Available Memory Bytes.
5.  Choose Add Metric. Select Percentage CPU.
6.  Choose Add Metric. Select Network Out Total.
7.  Select Line Chart and then choose Grid.
8.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
9.  In the list of virtual machines, select SYD-WS2022.
10. Under Monitoring choose Metrics.
11. On the Metric drop down, select Available Memory Bytes.
12. Choose Add Metric. Select Percentage CPU.
13. Choose Add Metric. Select Network Out Total.
14. Select Line Chart and then choose Grid.

### Exercise 3.3. Configure Azure Monitor for VM Data Collection Endpoint

1.  In the Azure Portal Search Bar, enter Monitor and select Monitor from the list of results.
2.  In the Monitor page, under Settings, choose Data Collection Endpoints.
3.  On the Data Collection Endpoints page, choose Create.
4.  Provide the following information:
    -   Endpoint Name: IaaSVMCollectionEndpoint
    -   Subscription: Your subscription:
    -   Resource Group: InfraResourceGroup
    -   Region: Choose the same region as that which is used by your resource group
5.  Choose Review + Create and then choose Create. This will create the data collection endpoint that you will use for monitoring IaaS VMs.

### Exercise 3.4. Configure Azure Monitor for VM Data Collection Rule

In this exercise, you will create a Data Collection Rule to collect a specific set of events from the Windows Server event logs.

1.  In the Azure Portal Search Bar, enter Monitor and select Monitor from the list of results.
2.  In the Monitor page, under Settings, choose Data Collection Rules.
3.  On the Data Collection Rules page, choose Create.
4.  On the Create Data Collection Rule page, configure the following settings and choose Next.
    -   Rule name: WinVMDCR
    -   Subscription: Your subscription
    -   Resource Group: InfraResourceGroup
    -   Region: The region in which your resource group is located.
    -   Platform type: Windows
    -   Data collection endpoint: IaaSVMCollectionEndpoint
5.  On the Resources page, choose Add Resources.
6.  On the Select a scope page, expand the InfraResourceGroup resource group and enable the SYD-WS2022 checkbox. Choose Apply.
7.  On the Create Data Collection Rule page, choose Next: Collect and deliver.
8.  On the Collect and deliver page of the Collect Data Collection Rule wizard, choose Add Data Source.
9.  On the Add Data Source page, select Windows Event Logs from the Data Source Type dropdown. Select the following checkboxes:
    -   Application: Critical, Error, Warning
    -   Security: Audit Failure
    -   System: Critical, Error, Warning
10. Choose Next: Destination.
11. On the Destination page, select the Azure Monitor Logs destination that is configured for your subscription and the LogAnalytics1 account and choose Add Data Source.
12. Choose Review + Create and then choose Create.

### Exercise 3.5. Modify VM Data Collection Rule to include IIS Logs

In this exercise, you will modify a Data Collection Rule to collect a specific set of events from the IIS log.

1.  In the Azure Portal Search Bar, enter Monitor and select Monitor from the list of results.
2.  In the Monitor page, under Settings, choose Data Collection Rules.
3.  Choose the WinVMDRC rule in InfraResourceGroup resource group.
4.  Under Configuration, choose Data Sources.
5.  On the Data Sources page, choose Add.
6.  On the Add Data Source page, select IIS Logs as the data source type.
7.  Choose Next: Destination.
8.  On the Destination page, ensure that the destination of Azure Monitor Logs and the log analytics workspace LogAnalytics1 is selected and then choose Add Data Source.

### Exercise 3.6. Configure Network Connection Monitor

1.  In the Azure Portal Search Bar, enter Network Watcher and select Network Watcher from the list of results.
2.  Under Monitoring, choose Connection Monitor.
3.  On the Connection Monitor page, choose Create.
4.  On the Basics page of the Create Connection Monitor wizard, provide the following information and choose Next: Test Groups:
    1.  Connection Monitor Name: IaaSVMTest
    2.  Subscription: Your Subscription
    3.  Region: Same region as resource group
5.  On the Add Test Group details page, enter the Test Group Name as VMTestGroup and choose Add Sources.
6.  On the Add Sources page, expand the InfraVNet and InfraSubnetNodes and enable the SYD-LINUX checkbox. Choose Add Endpoints.
7.  On the Add test group details page, choose Add Test Configuration. On the Add Test Configuration Page, provide the following details and accept the other defaults:
    1.  Test Configuration Nane: Default HTTP
    2.  Protocol: HTTP
    3.  Destination Port: 80
8.  Choose Add Test Configuration.
9.  On the Add test group details page, choose Add destinations.
10. On the Add destinations page, expand InfraVNet and then expand InfraSubnet and enable the SYD-WS2022 checkbox. Choose Add Endpoints.
11. On the Add test group details page, choose Add Test Group.
12. On the Test Groups page, choose Next: Workspaces.
13. On the Workspace page, choose Custom Workspace and ensure that LogAnalytics1 is selected. Choose Review and Create and then choose Create.
14. Doing this will crate the connection test as well as install the Network Watcher Extension on the Linux IaaS VM.
15. In the Azure Portal Search Bar, enter Network Watcher and select Network Watcher from the list of results.
16. Under Monitoring select Connection monitor.
17. Select IaaSVMTest and then choose View Logs.
18. Review the results of the log, checking the TestResult field. This should indicate (assuming that everything is configured correctly, that the Linux VM can send traffic to port 80 on the Windows Server VM with IIS installed and Port 80 open on the NSG rule)

### Exercise 3.7. Configure Performance Alerts and Action Groups

In this exercise you will configure a performance alert and an action to occur based on CPU utilization on a Linux IaaS VM.

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-LINUX.
3.  On the SYD-LINUX page, choose Alerts under Monitoring.
4.  On the Alerts page, choose Create and then choose Alert Rule.
5.  On the Condition page of the Create an Alert Rule wizard, set the Signal name to Percentage CPU. Note in the Preview page the default CPU utilization is under 10% on average.
6.  Accept the default settings and choose Next: Actions.
7.  On the Actions page, choose Create Action Group.
8.  On the Create Action Groups page, configure the following and then select Next: Notifications.
    1.  Subscription: Your subscription
    2.  Resource Group: InfraResourceGroup
    3.  Region: Global
    4.  Action Group Name: NotifyCPU
    5.  Display Name: NotifyCPU
9.  On the Notifications page, On the notification type drop down menu select Email Azure Resource Manager Role and set the name as Email Admin. Choose the pencil icon under Selected. On the Email Azure Resource Manager Role page, use the drop down and then select Owner. Choose OK. Choose Next: Details
10. On the Details page, enter the alert rule name HighCPU. Choose Review + Create and then choose Create.

## Section Four: IaaS VM Security

This section requires that you have completed section zero.

### Exercise 4.1. VM Vulnerability Assessment and Just In Time access

[As this takes some time to light up, it may be that Exercise 4.1 needs to be skipped of VMs aren’t present in the Vulnerability Assessment sections and the JiT sections. Workshop attendees can return to exercise 4.1 after they have completed section 5]

1.  In the Azure Portal search bar, type Defender for Cloud and select Defender for Cloud.
2.  On the Defender for Cloud menu, select Workload Protections under Cloud Security.
3.  Under Advanced Protection choose VM vulnerability assessment.
4.  On the Machines should have vulnerability assessment solution under Affected resources, select the checkboxes next to syd-ws2022 and syd-linux and then choose Fix.
5.  On the Choose a vulnerability assessment solution, select Microsoft Defender vulnerability Management and choose Proceed.
6.  On the Fixing Resources page, choose Fix 2 resources. Monitor the notification to verify remediation. This remediation will take several minutes to appear in the portal.
7.  In the Azure Portal search bar, type Defender for Cloud and select Defender for Cloud.
8.  On the Defender for Cloud menu, select Workload Protections under Cloud Security.
9.  On the Workload Protections page, under Advanced Protection, choose Just-In-Time VM Access.
10. On the Just-in-time VM access page, you may see SYD-WS2022 listed as you configured this setting indirectly when you configured Windows Admin Center. Select the Not Configured page and then select the checkbox next to SYD-LINUX (and SYD-WS2022 if it is not present). Chose Enable JIT on 1 (or 2) VMs.
11. On the JIT VM Access Configuration page, choose Save. Return to the Configured tab to verify that Just-in-time VM access is enabled for both SYD-WS2022 and SYD-LINUX.
12. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
13. In the list of virtual machines, select SYD-LINUX.
14. Under Connect choose Select under SSH using Azure CLI and then enable the I understand just-in-time policy checkbox and select Configure + Connect.
15. Verify that you are able to make an SSH connection using Cloud Shell to SYD-LINUX.
16. In the Azure Portal search bar, type Defender for Cloud and select Defender for Cloud.
17. On the Defender for Cloud menu, select Workload Protections under Cloud Security.
18. On the Workload Protections page, under Advanced Protection, choose Just-In-Time VM Access.
19. On the Configured VMs list, you will see the port 22 request for SYD-LINUX used by the account you used to connect to the Linux VM. Now that JiT is configured, all administrative connections will need to be requested and will be logged.
20. On the Defender for Cloud menu, select Workload Protections under Cloud Security.
21. On the Workload Protections page, under Advanced Protection, choose Adaptive Network Hardening.
22. On the Adaptive Network Hardening recommendations, choose the Healthy Resources tab. Both virtual machines will be listed here. Recommendations for these VMs based on analyzed traffic patterns.

### Exercise 4.2. Configure BitLocker Encryption and DMCrypt

In this exercise you will configure BitLocker Encryption for the Windows Server IaaS VM and DMCrypt for the Linux VM. To complete this exercise, perform the following steps:

1.  From the Azure Portal, open Cloud Shell
2.  Run the following command to list the keyvault name

    Az keyvault list --output table

3.  A name will be returned that will be in the format IaaSVMKVxxxxxxxxx where the string of xxxxxxx represents a number. Make note of this name as you’ll need to substitute it into commands in step 5 and step 7. You might also select it and copy it so that you can paste it into those commands using SHIFT-INS in step 5 and step 7.
4.  Type the following commands to verify that VM disk encryption is currently not enabled

    az vm encryption show --name SYD-WS2022 -g InfraResourceGroup

    az vm encryption show --name SYD-LINUX -g InfraResourceGroup

5.  Enable BitLocker encryption on the Windows Server IaaS VM with the following command:

    az vm encryption enable -g InfraResourceGroup --name SYD-WS2022 --disk-encryption-keyvault IaaSVMKVxxxxx

6.  Verify that the VM is encrypted by running the following command:

    az vm encryption show --name SYD-WS2022 -g InfraResourceGroup -o table

7.  Enable DM-Crypt encryption on the Linux IaaS VM with the following command:

    az vm encryption enable -g InfraResourceGroup --name SYD-LINUX --disk-encryption-keyvault IaaSVMKVxxxxx --volume-type ALL

8.  Verify that the VM is encrypted by running the following command:

    az vm encryption show --name SYD-LINUX -g InfraResourceGroup -o table

### Exercise 4.3. Configure Azure Software Update and Hotpatch

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
2.  In the list of virtual machines, select SYD-LINUX.
3.  On the SYD-LINUX page, choose Updates under Operations.
4.  On the Updates page, choose Go to Updates using Azure Update Manager.
5.  On the Updates page, choose Check for Updates.
6.  When queried on triggering an assessment, choose OK.
7.  After the assessment completes, click Refresh to review the total updates available. As you have run package updates on the Linux computer, you may not need to install any updates. If no updates are required, move on to Step 13. If updates are required, choose One-time update.
8.  On the Install one-time updates page, select SYD-LINUX and choose Next.
9.  On the Updates to install page, review the updates to be installed and choose Next.
10. On the Reboot Option page, select Reboot if required and choose Next.
11. On the Review + Install page, choose Install.
12. On the Updates page, choose History and then choose Refresh. This will display the progress of the update deployment. Wait until the status reaches Succeeded. You may have to wait several minutes and select Refresh multiple times.
13. On the SYD-LINUX updates page, choose Enable Now next to Period Assessment.
14. On the Change Update Settings page, set the Periodic Assessment drop down to Enable and choose Save.
15. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines.
16. In the list of virtual machines, select SYD-WS2022.
17. On the SYD-WS2022 page, choose Updates under Operations.
18. Chose Go to Updates using Azure Update Manager.
19. On the Updates page, choose Check for Updates.
20. When prompted whether you want to trigger an assessment, choose OK.
21. When the assessment completes, choose Refresh. If no updates are present, move on to Step 29.
22. Choose One-Time Update.
23. On the Machines page of the Install one-time updates wizard, select SYD-WS2022 and choose Next.
24. On the Updates page, review the updates to install and choose Next.
25. On the Reboot Option page, select Reboot if required and choose Next.
26. On the Review + Install page, choose Install.
27. On the Updates page, choose History and then choose Refresh. This will display the progress of the update deployment. Wait until the status reaches Succeeded. You may have to wait several minutes and select Refresh multiple times.
28. Choose Update settings.
29. On the Change Update Settings page, configure the following settings and choose Save.
    -   Periodic Assessment: Enable
    -   Hotpatch: Enable
30. On the SYD-WS2022 Updates page, choose Schedule Updates.
31. On the Create a maintenance Configuration Page, configure the following settings and choose Add a schedule.
    -   Subscription: Your subscription
    -   Resource Group: InfraResourceGroup
    -   Configuration Name: LabUpdateConfig
    -   Region: Your Region (determined in Exercise 0.1)
    -   Maintenance Scope: Guest
    -   Reboot Setting Reboot if Required
32. On the Add/Modify schedule page, provide the following information and choose Save.
    -   Start on 12/01/2023 12:00 AM
    -   Maintenance Windows 3 hours 55 minutes
    -   Repeats 1 Day
33. On the Create a maintenance Configuration page, choose Next: DynamicScopes.
34. On the Dynamic Scopes tab, choose Add a dynamic scope.
35. On the Add a dynamic scope page, next to subscriptions select your subscription. Choose Save. When warned that not all Azure Machines shown in the preview are configured to schedule updates choose Continue with supported machines only and choose Save.
36. On the dynamic scopes page, choose Next: Resources.
37. On the Resources page, choose Next: Updates.
38. On the Updates page, review the update classifications and choose Review + Create.
39. On the Review + Create page, wait for validation to occur and then choose Create.

## Section Five: IaaS VM BCDR

This section requires that you have completed Section Zero and Exercise 2.3

### Exercise 5.1. Perform Virtual Machine Backup

As you have enabled automanage in Exercise 2.3, each IaaS VM is already configured for backups. Whilst Automanage has been configured, it is unlikely that a backup has been taken as you have only enabled this service relatively recently. You can perform manual backups in addition to the backups managed through automanage at any time. You perform a manual backup by performing the following steps:

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-LINUX.
2.  On the SYD-LINUX page Search bar at the top of the list of items, type Backup.
3.  On the Welcome to Azure Backup for Azure VMs page, configure the following settings and choose Enable Backup.
    -   Recovery Services Vault: Select Existing
    -   Vault: Choose the DefaultBackupVault for your revion.
    -   Policy sub type: Standard
    -   Choose Backup Policy: DefaultPolicy
4.  On the Backup page, choose Backup Now.
5.  On the Retain Backup Until page, accept the default and choose OK.
6.  After the deployment completes, in the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-LINUX.
7.  On the SYD-LINUX page Search bar at the top of the list of items, type Backup.
8.  On the SYD-LINUX Backup page, choose Backup Now.
9.  On the Backup Now page, accept the default retention date and choose OK.
10. In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-WS2022.
11. On the SYD-WS2022 page Search bar at the top of the list of items, type Backup.
12. On the SYD-WS2022 page, note that backup is already configured by automanage but that the initial backup is pending and hasn’t been completed. Choose Backup Now.
13. On the Backup Now page, accept the default backup retention option and choose OK.

### Exercise 5.2: Configure Azure Size Recovery

Azure Site Recovery allows you to failover a virtual machine from one Azure region to another.

1.  In the Search bar of the Azure Portal, type Virtual Machines and then choose Virtual Machines. In the list of virtual machines, select SYD-WS2022.
2.  On the SYD-WS2022 page Search bar at the top of the list of items, type Disaster Recovery. Select Disaster Recovery.
3.  On the SYD-WS2022 Disaster Recovery Page, accept the default Target Region (which will be different from the current region the VM is located in and choose Next: Advanced Settings.
4.  On the Advanced Settings page, review the automatically assigned resource group, virtual network, and key vault names that will be created and choose Next: Review + Start Replication.
5.  On the Review + Start Replication page, choose Start Replication.
6.  Open the Azure Portal in a new browser tab and in the Search bar type Resource Groups. Select Resource Group.
7.  In the list of resource groups, select InfraResourceGroup.
8.  In the InfraResourceGroup ASR resource group, select the IaaSVMKVXXXXX key vault.
9.  On the IaaSVMKVXXXX key vault, select Access Policies.
10. On the Access Policies page, choose Create.
11. Add the following permissions and choose Next:
    -   Key permissions: Key Management Operations: List, Create, and Get
    -   Secret permissions: Secret Management Operations: Get, List and Set
12. On the Principal page, use Search to locate and choose the account you are signed in as (User1-XXXXXXX@ - which is listed in the Resources tab) and choose Select and then choose Next.
13. On the Review and Create page, choose Create.
14. In the Azure Portal Search bar type Resource Groups. Select Resource Group.
15. In the list of resource groups, select InfraResourceGroup-ASR.
16. In the InfraResourceGroup ASR resource group, select the IaaSVMKVXXXXX-asr key vault.
17. On the IaaSVMKVXXXX-asr key vault, select Access Policies.
18. On the Access Policies page, choose Create.
19. Add the following permissions and choose Next:
    -   Key permissions: Key Management Operations: List, Create, and Get
    -   Secret permissions: Secret Management Operations: Get, List and Set
20. On the Principal page, use Search to locate and choose the account you are signed in as (User1-XXXXXXX@ - which is listed in the Resources tab) and choose Select and then choose Next.
21. On the Review and Create page, choose Create.
22. When the assignment is complete, switch back to the tab on which the replication error is present and choose Start Replication. This will take up to 15 minutes.

### Exercise 5.3. Perform VM Test Failover

1.  When replication has successfully been enabled and the replication status is Healthy, click Refresh. An infrastructure view will display the virtual machine, Azure Site Recovery, and any managed disks.
2.  Click Refresh until the Status field lists Protected and the Test Failover item becomes available.
3.  On the SYD-WS2022 page, choose Test Failover.
4.  On the Test Failover page, set the Azure Virtual Network to InfraVNet-asr and choose Test Failover.
5.  Test Failover will take several minutes to complete.
6.  When the Status shiftst to Cleanup test failover pending, click Cleanup Test Failover pending. This will open a log showing the different failover jobs and their completion status.
7.  Return to the SYD-WS2022 Disaster Recovery Page and choose the Cleanup test failover item.
8.  On the Notes page enter “Failover Successful” and then enable the Testing is complete checkbox and choose OK.
9.  The test failover environment will be removed from the secondary datacenter.
