# gMSA on Azure Kubernetes Service workshop

Welcome to the gMSA on Azure Kubernetes Service workshop. This workshop was designed to provide customers an understanding of how gMSA can be deployed into AKS, how it works, and what are the necessary steps to get it configured.

## About gMSA, Windows containers, and Azure Kubernetes Service

Although Windows containers cannot be domain joined, they can still use Active Directory domain identities to support various authentication scenarios. To achieve this, you can configure a Windows container to run with a group Managed Service Account (gMSA), which is a special type of service account introduced in Windows Server 2012 and designed to allow multiple computers to share an identity without needing to know its password. Windows containers cannot be domain joined, but many Windows applications that run in Windows containers still need AD Authentication. To use AD Authentication, you can configure a Windows container to run with a group Managed Service Account (gMSA).

When gMSA for Windows containers was initially introduced, it required the container host to be domain joined, which created a lot of overhead for users to manually join Windows worker nodes to a domain. This limitation has been addressed with gMSA for Windows containers support for non-domain-joined container hosts. With Azure Kubernetes Service (AKS), you can enable gMSA on your Windows Server nodes, which allows pods running on Windows Server nodes to integrate with Active Directory.

### Objectives

This workshop has the following objectives:

- Provide an overview of gMSA on AKS, necessary components, and how to setup an environment for a Windows app that requires Active Directory authentication.
- Understand how the AksGMSA PowerShell module helps in the process of configuring gMSA on AKS.
- Understand the flow of configuring gMSA on AKS and how the multiple resources interact with each other.

## Pre-requisites

In order to perform the exercises in this workshop, you will need:

- An Azure subscription with role of Owner and limits to deploy an AKS cluster, an Azure VM, an Azure vNet, and an Azure Key Vault. 
- Internet access to open a browser session with the Azure Portal.

## Exercise 01 - Spinning up Azure environment

To get started, we need to spin up a new Azure environment with the necessary components for gMSA to work.
This include:

- An AKS cluster with a Windows node pool.
- An Azure vNet, separate from the managed environment of AKS. This is needed because we will be adding another VM (Domain Controller) to the same vNET.
- An Azure VM running Windows Server 2022, with Active Directory deployed and configured.

To deploy this environment:

1. Open a new browser session and navigate to [https://portal.azure.com](https://portal.azure.com)
2. Login to the Azure portal with your credential.
3. On the Azure portal, click the Cloud Shell button on the top right-corner.
4. If a message "You have no storage mounted" appears, select the subscription you want to perform this exercise with and click "Create Storage". No need to use an existing private virtual network if prompted.
5. Azure may ask how you want to create a storage account, select "We will create a storage account for you".
6. On the new Cloud Shell window, make sure "PowerShell" is the selected shell on the top left-corner. If Bash is selected, select PowerShell and wait for the Cloud Shell session to reload.
7. On the Cloud Shell window, select the upload/download files icon. Click Upload. On the window to select the file to upload, select the DeployAKS.ps1 file that accompanies this workshop.
8. On the Cloud Shell window, close the information banner that confirmed the file was uploaded and run "./DeployAKS.ps1".
9. Provide the following information when asked:

- Resource Group Name: GMSATestRG
- Resource Group Location: westus2 (You can use a different location depending on your geo-location)
- vNet Name: gmsavNet
- Subnet: gmsasubnet
- AKS Cluster name: GMSACluster
- Windows Server node credential username: Microsoft
- Windows Server node credential password: M1cr0s0ft@2024 (This password won't be used in this exercise)
- SSH Key passphrase: <Leave blank> (You can type and re-type a password if you choose. This won't be used in this exercise)

Note: The provisioning of the AKS cluster might take 5-10 min to complete.

- Windows Server node pool name: wspool

Note: The provisioning of the Windows Server node pool might take from 8-15 min to complete.

- Domain Controller name: DC01
- Domain Controller credential username: Microsoft
- Domain Controller credential password: M1cr0s0ft@2024 (This password WILL be used in this exercise. If you choose your own password, make sure to remember it later.)

You should see a message saying: Your Azure environment is now setup. Please continue from inside the VM that was just deployed.
Once the final resource is created, we can set up the Domain Controller.

## Exercise 2 - Configure Active Directory

Active Directory needs to be configured in order for our AD dependant application to work on Azure Kubernetes Service. In this exercise we will set up the DC01 VM created in the previous exercise.

1. On the Azure portal, click the search bar at the top and type "Resource Groups". Select the Resource Groups service in the results.
2. On the Resource groups view, click the GMSATestRG group.
3. On the GMSATestRG view, click the DC01 VM.
4. On the DC01 view, click the search bar on the left-hand side, and type "Run Command".
5. Click the Run command option from the results.
6. On the Run command view, click the "RunPowerShellScript" option.
7. On the Run Command Script side pane, paste the following code block:

```powershell
Install-windowsfeature -name AD-Domain-Services -IncludeManagementTools
$SecureSMAP = ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force
$Domain_DNSName = "contoso.local"
$Netbios = "CONTOSO"
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath "C:\Windows\NTDS" -DomainMode "WinThreshold" -DomainName $Domain_DNSName -DomainNetbiosName $Netbios -ForestMode "WinThreshold" -InstallDns:$true -LogPath "C:\Windows\NTDS" -NoRebootOnCompletion:$false -SysvolPath "C:\Windows\SYSVOL" -Force:$true -SkipPreChecks -SafeModeAdministratorPassword $SecureSMAP
```

8. Click Run. (This command will take a couple minutes to run. You will see a success message, but the VM will be automatically restarted. You can safely move to the next step)
9. Once the output shows the success message, close the Run Command Script and the other menus to return to the Azure portal home screen.

## Exercise 3 - Enable Azure Bastion to RDP into DC01 VM and take note of additional resources

With the Active Directory configured, you can now start the configuration of gMSA and deployment of your application to AKS. However, we need to perform the commands in teh Exercise 4 from inside the DC01. We will open an RDP connection to the DC01 and to achieve that in secure way, we will enable Azure Bastion:

1. On the Azure portal, click the Cloud Shell button on the top right-corner.
2. Once the session is open, paste the following PowerShell code block:

```powershell
$vnet = Get-AzVirtualNetwork -Name "gmsavNet" -ResourceGroupName "GMSATestRG"
Add-AzVirtualNetworkSubnetConfig -Name "AzureBastionSubnet" -VirtualNetwork $vnet -AddressPrefix "10.0.1.0/26" | Set-AzVirtualNetwork
$publicip = New-AzPublicIpAddress -ResourceGroupName "GMSATestRG" -name "gmsavNet-IP1" -location "WestUS2" -AllocationMethod Static -Sku Standard
New-AzBastion -ResourceGroupName "GMSATestRG" -Name "gmsavNet-bastion" -PublicIpAddressRgName "GMSATestRG" -PublicIpAddressName "gmsavNet-IP1" -VirtualNetworkRgName "GMSATestRG" -VirtualNetworkName "gmsavNet" -Sku "Standard"
```

3. Close the Azure Cloud Shell session.
4. On the Azure portal, click the search bar at the top and type "Subscriptions".
5. Click the Subscriptions icon in the search results.
6. Take note of the subscription ID for the subscription you are using for this workshop. This will be needed in the next exercise.
7. Close all views and return to the Azure portal home screen.

## Exercise 4 - Configure the AksGMSA PowerShell module on the DC01 VM

From this point forward in the workshop, we are assuming a regular environment of Kubernetes and Active Directory exists. Most companies will have a variation of the proposed environment in this workshop that ultimately are the requirements for gMSA to be deployed and an Active Directory dependant app to work. To configure gMSA in AKS (as well as the Active Directory), we will use the AksGMSA PowerShell module, which is available in the PowerShell Gallery.

Note: We are using the DC01 VM to run the AksGMSA PowerShell module. This is for simplicity of this workshop. In a real life/production environment, you should run this module in a machine that has internet connection and access to a domain controller.

1. On the Azure portal, select the search bar at the top and type "Virtual Machines".
2. On the Virtual Machines view, click the DC01 VM.
3. On the DC01 view, click the Networking Setting under Networking in the left-hand side menu.
4. Take note of the Private IP address provided to the DC01 VM. This will be used in an exercise later in this exercise.
5. On the DC01 view, click the Bastion option under Connect in the left-hand side menu.
6. On the DC01 | Bastion view, provide the username and password used to create the DC01 VM and click Connect.
7. On the new browser window opened by Bastion, the RDP session to the DC01 VM will be established. Make sure to select "Allow" in the pop-up window to allow copy/paste between your machine and the RDP session. Once the VM is logged on, close the Server Manager windows.
8. Click the Start menu, search Powershell, right-click the Windows PowerShell icon and select More -> Run as Administrator.
9. On the Powershell session, run "Install-Module -Name AksGMSA -Repository PSGallery -Force"
10. When asked to install and import the NuGet provider, type "Y" to install it.
11. Once the packages have been installed, type "Install-ToolingRequirements"
12. Once the required tooling has been installed, type "Connect-AzAccount -DeviceCode -Subscription "<SUBSCRIPTION_ID>"" (Replace the SUBSCRIPTION_ID with the subscription ID you took note in the previous exercise).
13. On your own machine, open a new browser and type <https://microsoft.com/devicelogin>. Follow the login process using the code provided in the PowerShell session in the DC01 VM.
14. Once you approved the login, close the browser session and return to the PowerShell session in the DC01 Bastion tab on your browser. You should see a success authentication with the information on the Account, SubscriptionName, TenantID, and Environment.
15. On the PowerShell session, type "az login --use-device-code". (Repeat the process of logging in and authenticating just like the previous step. This is necessary to authenticate both the Az PowerShell module and the Az CLI).
16. Once the authentication for Az CLI has been completed, you should see a list of all subscription your account has access to. Type "az account set --subscription "<SUBSCRIPTION_ID>"" (Replace the SUBSCRIPTION_ID with the subscription Id you took note in the previous exercise).

The AksGMSA module is now installed and the Azure connectivity is established. Next, we will setup the parameters for the module to use and configure the AKS cluster and all other resources.

17. Run the command "$params = Get-AksGMSAParameters".
18. Provide the following parameters for the module:

- AKS cluster name: GMSACluster
- AKS cluster RG name: GMSATestRG
- AKS Windows node pools: wspool
- AD Domain DNS Server: <DC01_VM_IP_Address> (replace the DC01_VM_IP_Address with the internal IP address of the DC01 VM)
- AD Domain FQDN: contoso.local
- gMSA name: gmsa
- gMSA Domain user name: Microsoft
- gMSA Domain user password: M1cr0s0ft@2024 (If you used a different password, replace this with that password)
- Azure location: westus2 (If you used a different location, replace this with that location)
- AKV name: gmsa<variable>test (Replace variable for something unique. Azure Key Vault names need to be unique to all key vaults in Azure. We suggest concatenating your name, a date, etc. Use lowercase letters and numbers and keep it all together, no spaces.)
- AKV secret name: gmsasecret
- Azure MI name: gmsami
- gMSA spec name: credspec
- Local logs directory: (leave blank)
- AD Domain controller address: <DC01_VM_IP_Address> (replace the DC01_VM_IP_Address with the internal IP address of the DC01 VM)
- Domain admin user name: Microsoft
- Domain admin user password: M1cr0s0ft@2024 (If you used a different password, replace this with that password)

19. To validate the connection to the AKS cluster, run the following PowerShell code block:

```powershell
Import-AzAksCredential -Force `
-ResourceGroupName $params["aks-cluster-rg-name"] `
-Name $params["aks-cluster-name"]
```

20. run "kubectl get nodes -o wide"

You should see a list of 3 nodes from the AKS cluster - two Linux nodes and one Windows Server 2022 node.

## Exercise 5 - Deploy gMSA on AKS and configure AD and Azure resources

From now on, the AksGMSA module is setup with the variables needed to configure all the resources for gMSA to work. The module was created to ease the process of deploying all the satellite resources and configuring all settings required for AKS nodes to communicate with AD. As a user of the module, all you have to do is to copy and paste the commands to get the services configured.

Run the following commands in this order:

1. Confirm the AKS cluster has gMSA feature properly configured or configure it:

```powershell
Confirm-AksGMSAConfiguration `
-AksResourceGroupName $params["aks-cluster-rg-name"] `
-AksClusterName $params["aks-cluster-name"] `
-AksGMSADomainDnsServer $params["domain-dns-server"] `
-AksGMSARootDomainName $params["domain-fqdn"]
```

If the gMSA feature is not enabled, you will be asked to enable and configure it for the cluster. Type Y to configure it. This process might take 5-10 min to complete. Once completed, you can run the command again to confirm gMSA has been configured in the cluster.

2. Configure your Active Directory environment:

At this point, we have to perform a one time preparation in the Domain Controller to enable the KDSRootKey. This service allows AD to rotate secrets for the service accounts. To perform that you can run:

```powershell
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

Now, we just need to create the necessary gMSA and standard user domain. (For this workshop, we are using the same account "Microsoft" for everything. In a real environment, you should use a different account for each action.)

```powershell
New-GMSADomainUser `
-Name $params["gmsa-domain-user-name"] `
-Password $params["gmsa-domain-user-password"] `
-DomainControllerAddress $params["domain-controller-address"] `
-DomainAdmin "$($params["domain-fqdn"])\$($params["domain-admin-user-name"])" `
-DomainAdminPassword $params["domain-admin-user-password"]
```

```powershell
New-GMSA `
-Name $params["gmsa-name"] `
-AuthorizedUser $params["gmsa-domain-user-name"] `
-DomainControllerAddress $params["domain-controller-address"] `
-DomainAdmin "$($params["domain-fqdn"])\$($params["domain-admin-user-name"])" `
-DomainAdminPassword $params["domain-admin-user-password"]
```

3. Setup Azure Key Vault and Azure user-assigned Managed Identity

Azure Key Vault (AKV) will be used to store the credential used by the Windows nodes on AKS to communicate to the Active Directory Domain Controllers. A Managed Identity (MI) will be used to provide proper access to AKV for your Windows nodes.

```powershell
New-GMSAAzureKeyVault `
-ResourceGroupName $params["aks-cluster-rg-name"] `
-Location $params["azure-location"] `
-Name $params["akv-name"] `
-SecretName $params["akv-secret-name"] `
-GMSADomainUser "$($params["domain-fqdn"])\$($params["gmsa-domain-user-name"])" `
-GMSADomainUserPassword $params["gmsa-domain-user-password"]
```

The command above created the Azure Key Vault. AKV will be used to securely store the user account for the Windows nodes. This is because the Windows nodes on AKS are not domain joined and don't have a computer account. Now let's create Managed Identity so the Windows nodes can access AKV.

```powershell
New-GMSAManagedIdentity `
-ResourceGroupName $params["aks-cluster-rg-name"] `
-Location $params["azure-location"] `
-Name $params["ami-name"]
```

The command above created the Azure Managed Identity. This MI will be associated to the Windows node pool.

```powershell
Grant-AkvAccessToAksWindowsHosts `
-AksResourceGroupName $params["aks-cluster-rg-name"] `
-AksClusterName $params["aks-cluster-name"] `
-AksWindowsNodePoolsNames $params["aks-win-node-pools-names"] `
-VaultResourceGroupName $params["aks-cluster-rg-name"] `
-VaultName $params["akv-name"] `
-ManagedIdentityResourceGroupName $params["aks-cluster-rg-name"] `
-ManagedIdentityName $params["ami-name"]
```

The command above associated the Managed Identity to the Windows node pool and provided it access to the Azure Key Vault. Now, the final configuration is to deploy a credential spec and RBAC.

```powershell
New-GMSACredentialSpec `
-Name $params["gmsa-spec-name"] `
-GMSAName $params["gmsa-name"] `
-ManagedIdentityResourceGroupName $params["aks-cluster-rg-name"] `
-ManagedIdentityName $params["ami-name"] `
-VaultName $params["akv-name"] `
-VaultGMSASecretName $params["akv-secret-name"] `
-DomainControllerAddress $params["domain-controller-address"] `
-DomainUser "$($params["domain-fqdn"])\$($params["gmsa-domain-user-name"])" `
-DomainUserPassword $params["gmsa-domain-user-password"]
```

The command above created a credential spec on the Windows nodes, and set up the RBAC resources.
At this point, the deployment of gMSA is complete on the AKS cluster. You can now deploy a Windows application that is Active Directory dependent.

## Exercise 6 - Validate the deployment of gMSA on AKS

Once you configure gMSA on AKS with the PowerShell module, your application is ready to be deployed on your Windows nodes on AKS. However, you might want to further validate that the configuration is set-up correctly. The AksGMSA PowerShell module provides mechanisms to achieve that. First, we will verify the Active Directory connectivity. Then we will verify that the Windows hosts are being issued kerberos tickets from the Domain Controller. Finally, we will validate that Windows node pools (and only the that node pool) has access to Azure Key Vault.

1. To validate the connectivity between the AKS cluster and the Domain Controller, you can run:

```powershell
Confirm-AksADDCConnectivity `
-AksResourceGroupName $params["aks-cluster-rg-name"] `
-AksClusterName $params["aks-cluster-name"] `
-AksWindowsNodePoolsNames $params["aks-win-node-pools-names"] `
-DomainDnsServer $params["domain-dns-server"] `
-RootDomainName $params["domain-fqdn"]
```

Note: If you get an error message that the pod is not ready, run the command again.
The command above performs an nslook from the Windows pod to the domain, with the output providing the Active Directory domain configuration as you would expect from a domain-joined machine. This confirms that the Windows pod and Windows node were able to successfully connect resolve the DNS for the domain, which in turn allows them to communicate with the Domain Controller.

2. To validate the credential spec and check the kerberos ticket issued to the Windows node, you can run:

```powershell
Start-GMSACredentialSpecValidation `
-SpecName $params["gmsa-spec-name"] `
-AksWindowsNodePoolsNames $params["aks-win-node-pools-names"]
```

Note: If you get an error message that the pod is not ready, run the command again.
The command above also performs an nslookup, but it also checks for the kerberos ticket issued to the Windows node from the Domain Controller. It confirms the Windows node was not only able to communicate with the DC, but able to authenticate against it.

3. To check which node pools in the AKS cluster have access to Azure Key Vault, you can run:

```powershell
Get-AksAgentPoolsAkvAccess `
-AksResourceGroupName $params["aks-cluster-rg-name"] `
-AksClusterName $params["aks-cluster-name"] `
-VaultResourceGroupNames $params["aks-cluster-rg-name"]
```

The command above checks for the permissions on the Azure Key Vault against all node pools in the AKS cluster. It shows a table with the IsSuthorized collumn with True/False. Only the appropriate Windows node pools that run the Windows pods with the Windows applications should have access to AKV.

## Exercise 7 - Deploy IIS with Windows authentication enabled

Once the AKS cluster is configured with gMSA and has been properly validated, we can deploy an application. In this workshop, we will deploy a sample, minimal IIS configuration to see the authentication from the Windows pod with Active Directory.

1. Run the following Powershell code block:

```powershell
Get-GMSASampleApplicationYAML `
-SpecName $params["gmsa-spec-name"] `
-AksWindowsNodePoolsNames $params["aks-win-node-pools-names"] | kubectl apply -f -
```

The above command will deploy a YAML specification to the AKS cluster. That specification deploys a Windows pod with IIS enabled, and a simple page that uses Windows Authentication.

2. Run the command "kubectl get pods -w". Wait until the pod status for the "credspec-demo-XXXXXX" changes from ContainerCreating to Running.
3. Run the command "kubectl get service". Copy the external IP address of the service "credspec-demo" deployed for the sample application.
4. Open a browser session and navigate to http://<IP_Address> (Replace IP_address with the IP address you copied in the previous step).
5. When asked for credentials, type the username and password for the credentials we used to deploy both the DC01 VM and domain. (This should be the username Microsoft and password M1cr0s0ft@2024, unless you changed these settings)

Once you log in, the webpage will show the information about the logged user. This confirms the Windows pod was able to communicated with the Domain Controller to authenticate and authorize the access.

## Exercise 8 - Clean up

To clean up the environment:

1. Open a browser and navigate to the Azure portal at <https://portal.azure.com>.
2. On the Azure portal, click the Azure Cloud Shell icon on the top right-hand corner.
3. Once the Cloud Shell session opens, run the following PowerShell code block:

```powershell
$RG_Name = Read-Host -Prompt "Please provide the Resource Group Name you want to delete"
Remove-AzResourceGroup -Name $RG_Name -Force
```

There might be other resource groups created along with the resources above (Such as Network Watcher, Cloud Shell storage account, etc.) Please double check your subscription for remaining resources.
