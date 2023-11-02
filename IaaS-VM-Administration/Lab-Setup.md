```Cloud Shell PowerShell

# Enter cloudshell and create a file named deploy.ps1. Paste in the contents of this file and then execute.

# Assign Variables
$vmPassword = "YourAmazingSecurePasswordHereThatYouRemember"
$rgname = "InfraResourceGroup"
# Set Lab Location to appropriate Datacenter Name
$lablocation = "westus" 
$randomNumber = Get-Random -Minimum 100000 -Maximum 999999
$keyVaultName = "IaaSVMKV"
$keyVaultNameWithRandom = "${keyVaultName}${randomNumber}"
$bootDiagName = "iaasbootdiag"
$bootDiagNameWithRandom = "${bootDiagName}${randomNumber}"

# Create Resource Group
az group create --name $rgname --location $lablocation

# Create Virtual Network
az network vnet create --name InfraVNet --resource-group $rgname --address-prefix 10.0.0.0/16 --subnet-name InfraSubnet --subnet-prefixes 10.0.0.0/24

# Create Log Analytics Workspace
az monitor log-analytics workspace create --workspace-name LogAnalytics1 --resource-group $rgname --location $lablocation

# Create WS 2022 Datacenter Azure Edition VM
az vm create --resource-group $rgname --name SYD-WS2022 --location $lablocation --image MicrosoftWindowsServer:WindowsServer:2022-datacenter-azure-edition-hotpatch:20348.2031.231006 --size Standard_D2s_v3 --vnet-name InfraVNet --subnet InfraSubnet --admin-username prime --admin-password $vmPassword

# Create Ubuntu Linux VM
az vm create --resource-group $rgname --name SYD-LINUX --location $lablocation --image Ubuntu2204  --size Standard_D2s_v3 --vnet-name InfraVNet --subnet InfraSubnet --admin-username prime --admin-password $vmPassword

# Create Keyvault for Iaas VM Encryption
az keyvault create --name $keyVaultNameWithRandom --resource-group "$rgname" --location $lablocation --enabled-for-disk-encryption

# Create Storage Account for Boot Diagnostics
az storage account create --name $bootDiagNameWithRandom --resource-group $rgname --location $lablocation --sku Standard_LRS --kind StorageV2

```