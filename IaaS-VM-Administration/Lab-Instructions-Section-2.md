# Optimizing Virtual Machine Workloads on Azure.

## Section Two: Manage IaaS VM Configuration

Section two only requires that you have performed section zero and the parts of exercise 1.3 where you added an extra disk to SYD-WS2022.

### Exercise 2.1. Enable Boot Diagnostics

In this exercise, you will enable boot diagnostics on Azure IaaS Virtual Machines.

You can see a quick audio free video showing the lab steps here: [Exercise 2.1 Demo Video](https://youtu.be/I8XbtRHL7k0)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-WS2022**.
3. Under **Monitoring**, choose **Diagnostic settings**.
4. On the **Install Diagnostics Extension** page, under **Diagnostics Storage Account**, choose the Storage Account named **iaasbootdiagnosticsXXXX** (where XXXX is a unique number) and choose **Enable Guest-level Monitoring**.
5. Once the installation completes, under **Help**, choose **Boot Diagnostics**.
6. On the **Boot Diagnostics** page, choose **Settings**.
7. On the **Boot Diagnostics Settings** page, choose **Enable with Custom Storage Account** and on the **Diagnostics Storage Account** drop down select **iaasbootdiagnosticsXXXX**. Choose **Apply**.
8. When the configuration completes, In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
9. In the list of virtual machines, select **SYD-LINUX**.
10. Under **Help** on the SYD-LINUX properties page, choose **Boot Diagnostics**.
11. On the **Boot Diagnostics** page, choose **Settings**.
12. On the **Boot Diagnostics** page, choose **Enable with custom storage account** and choose the Storage Account named **iaasbootdiagnosticsXXXX** and select **Apply**.
13. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
14. In the list of virtual machines, select **SYD-WS2022**.
15. Under Help choose Boot Diagnostics. You should now see an image of a locked Windows Server screen displaying the time and date. If a screen shot is not present, choose **Refresh**.
16. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
17. In the list of virtual machines, select **SYD-LINUX**.
18. Under Help choose Boot Diagnostics. You should not see an image of a Linux system waiting for sign on. If a screen shot is not present, choose Refresh.

### Exercise 2.2. Configure and administer Windows Server with Windows Admin Center

In this exercise, you will configure a Windows Server IaaS VM to be managed from the Azure portal using Windows Admin Center.

You can see a quick audio free video showing the lab steps here: [Exercise 2.2 Demo Video](https://youtu.be/k6gFW-mnJKk)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-WS2022**.
3. Under **Connect** choose **Windows Admin Center**.
4. On the Windows Admin Center page, review the inbound port and enable the **Open an outbound port for Windows Admin Center** to install and choose **Install**.
5. After installation completes, select **Access control (IAM)** and then choose **Add role assignment**.
6. Choose **Windows Admin Center Administrator Login** and choose Next.
7. Choose Select members and use Search to locate and choose the account you are signed in as (**User1-XXXXXXX@** - which is listed in the Resources tab) and choose **Select**.
8. Choose **Review and Assign** and then choose **Review and Assign** again.
9. On the SYD-WS2022 page, choose **Connect** under **Settings**.
10. On the **Connect** page, under **Windows Admin Center**, choose **Connect In Browser**. On the **Windows Admin Center** pane, choose **Configure**. (*If you encounter an error, choose the separate Windows Admin Center item in the VM menu pane and choose the Public IP address option. If that doesn’t work, you may also need to restart the IaaS VM*)
11. In the **Windows Admin Center Tools** menu, choose **Storage**. You will see a set of virtual disks. One of these will not be initialized as you created and attach the disk in Exercise 1.3.
12. Select the not initialized and choose **Initialize Disk**. Initialize the disk with the GPT partition style and choose **Submit**.
13. Select the newly initialized disk and choose **Create Volume**.
14. On the Create Volume page, configure the following settings and choose Submit:
- Drive Letter: **O**
- Volume Label: **Data Disk**
- File System: **ReFS**
- Allocation unit size: **4096**
- Use maximum size: **Enable checkbox**
15. On the **Storage** page, choose **Volumes** and review the volumes attached to SYD-WS2022.
16. In **Windows Admin Center**, choose **Files & File Sharing**, and then choose **File Shares**.
17. Choose **File Server Settings**.
18. Choose the following settings and choose Save.
- SMB signing: **Required**
- SMB 3 encryption: **Required from all clients that support it**
- Request SMB compression: **Enabled**
19. On the **Windows Admin Center Tools** menu, choose **Certificates**. Expand **Overview** and view the **Expired certificates**.
20. On the **Windows Admin Center Tools** menu, choose **Firewall**.
21. On the **Firewall** page, choose **Incoming rules**. Locate the **World Wide Web Services (HTTP Traffic-In)** rule and verify that it is set to Allowed. This rule was configured when you installed IIS.
22. On the **Windows Admin Center Tools** menu, select **Roles & Features**.
23. On the list of **Roles and Features**, expand **File and Storage Services** and then expand **File and iSCSI Services**.
24. Select **Data Deduplication** and then choose **Install** under **Roles and Features** near the top of the page.
25. Choose **Yes** on the **Install Roles and Features** page.
26. On the **Tools** menu, choose **Remote Desktop**.
27. On the Remote Desktop login credentials page, provide the following credentials:
- Username: **.\\Prime**
- Password: **P@ssw0rdP@ssw0rd**
- Automatically connect with the certificate presented by this machine: **Enabled**
28. Choose **Confirm** when prompted and then choose **Connect**.
29. You will be connected to a Remote Desktop session over the browser and the Windows Server Desktop Experience will be displayed. Click **Cancel** to Dismiss the Shutdown Event Tracker dialog box (this will be present due to reboots from earlier exercises). If you have performed this part of the lab after section 1, Internet Information Services will be shown as a node in Server Manager.
30. Through the web based Remote Desktop session, right click on the Start hint and choose **Windows PowerShell (Admin)**
31. At the PowerShell prompt type the following command and press Enter to view the computer name:
+++Hostname+++

32. To view the configured IP address settings, type the following command and press Enter:

**Get-NetIPAddress**

33. Verify that Deduplication has installed by running the following command:

**Get-WindowsFeature FS-Data-Deduplication**

34. Choose Disconnect under Remote Desktop to end the Remote Desktop Session.
35. On the Windows Admin Center Tools menu, choose Processes. This will allow you to see the processes that are currently running on SYD-WS2022. Locate the HotPatch process. This process is new to the Azure Edition of Windows Server 2022. You will learn more about HotPatch later in this lab.
36. On the Windows Admin Center Tools menu, choose Registry. This tool allows you to view and modify the registry on the Windows Server computer remotely without signing on directly to the Windows Server computer. On the Registry page, expand the following hive **HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion** and locate the **ProductName** key. This informs you of the version of Windows Server.
37. On the Windows Admin Center Tools menu, select Packet Monitoring. On the Consent to installing additional software page, enable both the I Agree To The License Terms And Consent To Installing The Software Above and Automatically Open Packet Monitoring Logs After Logging Has Been Stopped checkboxes and choose Consent To Installation.
38. On the Packet Monitoring page, choose New Capture.
39. On the Filter the captured packets by PacketMon page choose Add. 
40. On the Capture Filters enter IP Address 1 as 10.0.0.40 (the IP address you configured in Exercise 1.1) and Port 1 as 80 and then choose Start. 
41. Open a new browser tab and navigate to the public IP address of the IIS web server hosted on the Windows Server IaaS VM. 
42. Switch back to the original browser tab and choose Stop under Packet Monitoring. The results of the packet capture will be displayed, showing traffic connecting to the VM from a remote IP address and interacting with the IIS web server.
43. On the Windows Admin Center Tools menu, choose Services. On the Services page, choose Status at the top of the Status column to sort services into Running and Stopped. Select the Spooler service and then choose Restart. On the **Restart Service** pop up, choose **Yes**.

### Exercise 2.3. Configure Automanage for Azure VM

In this exercise you will configure Automanage for SYD-WS2022. Automanage allows you to automatically configure a number of Azure IaaS services that support Azure VMs without having to configure each on and individual basis.

You can see a quick audio free video showing the lab steps here: [Exercise 2.3 Demo Video](https://youtu.be/biEZMJLXvdY)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-WS2022**.
3. On the SYD-WS2022 page, choose **Automanage** under **Operations**.
4. On the **Automanage** page, choose **Custom Profile**.
5. On the **Custom Profile** drop down select **Create New**.
6. On the **Create A Custom Profile Page**, enter or confirm the following settings and choose Create:
- Name: **AutomanageLab**
- Subscription: **Your Subscription**
- Resource Group: **InfraResourceGroup**
- Region: **Your region**
- Enable Backup: Ensure checkbox is checked.
- Frequency: **Daily**
- At: **1:00 AM**
- Timezone: **UTC**
- Retain Instant Recovery Snapshots for **2**
- Retention of daily backup point: **180 days**
- Enable Microsoft Antimalware: Ensure checkbox is checked.
- Enable Real-time protection: Ensure checkbox is checked.
- Enable run a schedule scan: Ensure checkbox is checked.
- Scan type: **Full**
- Scan Day: **Sunday**
- Scan time: **120**
- Enable Machine Insights Monitoring: Ensure checkbox is checked.
- Enable security baseline: Uncheck checkbox. (You will enable this manually later)
- Enable update management: Uncheck checkbox (You will enable this manually later)
- Enable change tracking and inventory: Uncheck checkbox (You will enable this manually later)
- Enable Azure Boot Diagnostics: Uncheck checkbox (you already performed this step in Exercise 2.1)
- Enable Windows Admin Center: Uncheck checkbox (you already performed this step in Exercise 2.2)
7.  With the Custom Profile set to **AutomanageLab**, choose **Enable**.

### Exercise 2.4. Configure Azure Policy for IaaS VM

In this exercise you will use Azure Policy to evaluate several elements of the configuration of IaaS VMs. Policy can be a useful tool to allow you to determine if your IaaS VMs are configured correctly or require manual or automatic remediation.

You can see a quick audio free video showing the lab steps here: [Exercise 2.4 Demo Video](https://youtu.be/1HG6jmKDoE4)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. On the SYD-LINUX page, select **Policies** under **Operations**.
4. On the SYD-LINUX Policies page, choose **Assign Policy**.
5. On the **Assign Policy** page, select the ellipsis (…) next to **Policy definition**.
6. On the list of Available Definitions, type **SSH** in the Search textbox and select the **Authentication to Linux machines should require SSH keys** checkbox and choose Add.
7. On the **Assign Policy** page, choose **Review and Create** and then choose **Create**.
8. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
9. In the list of virtual machines, select **SYD-WS2022**.
10. On the SYD-WS2022 Policies page, search for Policies and choose **Assign Policy**.
11. On the **Assign Policy** page, select tie ellipsis (…) next to **Policy definition**.
12. On the Available Definitions page, in the search bar type **Certificates**. Select the **Certificates using RSA cryptography should have the specified minimum key size** policy and choose **Add**. Choose **Next** on the **Basics** page of the **Assign Policy** wizard.
13. On the **Advanced** page of the **Assign Policy** wizard, choose **Next**.
14. On the **Parameters** page, use the drop down menu and select **2048**. Choose **Review and Create** and then choose **Create**.
15. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
16. In the list of virtual machines, select **SYD-LINUX**. On the SYD-LINUX Policies page, choose Refresh until the Compliance State changes to **Non-compliant**. This is because SYD-LINUX allows sign in via username and password to simplify the operation of this lab exercise. You can come back to this step later after you have completed exercise 2.6 if you don’t want to wait for the compliance state to change.
17. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
18. In the list of virtual machines, select **SYD-WS2022**. On the SYD-WS2022 Policies page, choose Refresh until the Compliance State of the Certificates using RSA policy changes to Compliant. This may take several minutes. Note that the Authentication to Linux Machines policy will be listed as Compliant because SYD-WS2022 is not running the Linux operating system. You can come back to this step later after you have completed exercise 2.6 if you don’t want to wait for the compliance state to change.

### Exercise 2.5. Connect via Serial Connection to IaaS VM

The serial console connection allows you to make a special administrative connection to an IaaS VM when there may be some sort of fault with the network configuration. Serial Console allows you to perform actions against the VM from a command line environment even if the network connection doesn’t work. You can use this for troubleshooting or diagnosis purposes. You can make serial console connections to Linux IaaS VMs and Windows Server IaaS VMs.

You can see a quick audio free video showing the lab steps here: [Exercise 2.5 Demo Video](https://youtu.be/AXhXpbMrO3M)

1. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
2. In the list of virtual machines, select **SYD-LINUX**.
3. Under Help choose Serial Console.
4. When prompted, login with the following credentials:
- Login: **prime**
- Password: **P@ssw0rdP@ssw0rd**
5.  Type the following command to see information about your login

**w $(whoami)**

6. Type the following to verify that SYD-LINUX has network connectivity.

+++ping www.azure.com+++

7. Use **CTRL-C** to stop the output of the ping command.
8. Type **exit** to sign off from the serial console connection to the Linux IaaS VM.
9. On the **SYD-LINUX** page Under **Help** choose **Boot Diagnostics** and then choose **Serial log**.
10. You should see a log record of the commands you issued when connected via the Serial connection.
11. In the **Search bar of the Azure Portal**, type **Virtual Machines** and then choose **Virtual Machines**.
12. In the list of virtual machines, select **SYD-WS2022**.
13. Under Help choose Serial Console.
14. Once the console connection is established, At the SAC\> prompt, type **CMD**
15. The message will return that a new channel named Cmd001 has ben created.
16. At the SAC\> prompt, type

+++ch -sn Cmd0001+++

17. Press Enter to view the channel. You will be prompted to log on to the computer. Provide the following credentials:

-Username: **prime**

-Domain: **\<Press Enter to provide no domain information\>**

-Password: **P@ssw0rdP@ssw0rd**

13. You will be presented with the Windows Server command prompt. Type the following command to verify network connectivity:

+++Ping www.azure.com+++

14. Type sconfig to load the Windows Server command line configuration utility

+++sconfig+++

15. On the sconfig menu, type **8** to view the network settings.
16. Type the number associated with the network adapter (it will be 1 or 2).
17. Press **Enter** to return to the main menu and then **12** to log off the user.
18. When prompted if you want to log off, choose **Y**.
19. Type **exit** to return to the SAC environment.
20. On the **SYD-WS2022** page Under **Help** choose **Boot Diagnostics** and then choose **Serial log**.
21. You should see a log record of the commands you issued when connected via the Serial connection.

===
Choose Next to go to Section 3