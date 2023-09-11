# LAB160: How to simplify and integrate security in the SDLC!
**Objective**

In this lab, you will learn how to enable Microsoft Defender for DevOps in your pipelines and leverage its capabilities to improve the security of your cloud native applications and shift-left remediation of security vulnerabilities and misconfigurations.

For the purpose of this lab, the ADO orgnanization has already been onboarded to Microsoft Defender for Cloud, and Microsoft Defender for DevOps has been enabled.

## Prerequisite ##
- Create a resource group (In this lab, we will use RG1) on Azure portal: https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal

## Exercise 1: Configure Defender for DevOps in your pipeline and remediate security issues found in code.
**Objective**: Configure the Microsoft Security DevOps (MSDO) task in the YAML file of a build pipeline and remediate vulnerabilities and misconfigurations identified in your web application.

1. Connect to the Virtual Machine using the credentials you have in the Resources tab. Be sure to allow clipboard access for the VM, if prompted.
2. Navigate to Azure DevOps project using this link - https://dev.azure.com, and login using your Azure credentials once prompted and create a private project called **DevSecOps Lab**. 
3. Create a Service Connection by opening **Project settings** at the bottom left.
4. Click on **Service Connection** under **Pipelines**.
5. Click on **New service connection** on the top right on the screen, select **Azure Resource Manager**, click on **Next**, and **Next** again.
6. Select your **resource group**, add a name as follows: **SC1**, and click **Save**. Please do **NOT** grant access permission to all pipelines.
7. Navigate to **Pipelines** (the 4th icon on the left side menu), and open **Pipelines**.
8. Click on **New pipeline** on the top right of the page, and create a new pipeline that connects to your ADO repository.
9. When configuring your pipeline, select 'Azure Repos Git', 'Repo1' for the repository, and ‘Starter pipeline’.
10. Replace the pipeline YAML file you see on your screen with the one below. Change the file name to **msdo.yml**.

json
```
trigger:
- main

steps:
- task: MicrosoftSecurityDevOps@1
  displayName: 'Microsoft Security DevOps'
```

11. Save and run the pipeline. Commit the file directly to the main branch.
12. Create a branch protection by opening the project settings.
13. Navigate to Repositories under **Repositories, Select your repository**, and open **Policies**. Please do **NOT** create the branch policy on the project-level.
14. Click on the branch **main** in **Branch Policies**.
15. Click on the icon + in **Build Validation** to add a build policy, select the pipeline called **Repo1**. Leave the rest of the configuration as default. Hit **Save**.
16. Navigate back to **Repos** and create a new branch by opening the dropdown menu **main**, and click + **New Branch**.
17. Call your branch **Branch1**, and click **Create**.
18. Click the three dots next to your Repo name and create a new file. Call the file **ContosoFinanceAppProd.json**.

Enter the following code:
json
```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "webAppName": {
            "type": "string",
            "defaultValue": "[concat('WebApp-', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "Web app name"
            }
        },
        "appServicePlanName": {
            "type": "string",
            "defaultValue": "[concat('AppServicePlan-', uniqueString(resourceGroup().id))]",
            "metadata": {
                "description": "App service plan name"
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Web/serverfarms",
            "apiVersion": "2021-02-01",
            "name": "[parameters('appServicePlanName')]",
            "location": "[resourceGroup().location]",
            "sku": {
                "name": "F1",
                "capacity": 1
            },
            "kind": "app"
        },
        {
            "type": "Microsoft.Web/sites",
            "apiVersion": "2021-02-01",
            "name": "[parameters('webAppName')]",
            "location": "[resourceGroup().location]",
            "dependsOn": [
                "[resourceId('Microsoft.Web/serverfarms', parameters('appServicePlanName'))]"
            ],
            "properties": {
                "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('appServicePlanName'))]",
                "httpsOnly": false,
                "minTlsVersion": "1.1"
            }
        }
    ]
}
```

19. **Commit** the file.
20. Create a new file in the same branch, called **MyWebApp.html**.
Copy the following code in your new file:
json
```
<!DOCTYPE html>
<html>
<head>
  <title>Welcome to Lab160</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f8f8f8;
    }

    #header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 20px;
      background-color: #333;
      color: #fff;
    }

    #header h1 {
      font-size: 48px;
      margin: 0;
    }

    #header button {
      background-color: #fff;
      color: #333;
      border: none;
      padding: 10px 20px;
      border-radius: 5px;
      font-size: 24px;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    #header button:hover {
      background-color: #333;
      color: #fff;
    }

    #content {
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 500px;
    }

    #content h2 {
      font-size: 36px;
      margin: 0;
      text-align: center;
      color: #333;
    }

    #content p {
      font-size: 24px;
      line-height: 1.5;
      margin: 20px 0;
      max-width: 800px;
      text-align: center;
      color: #666;
    }

    #content button {
      background-color: #333;
      color: #fff;
      border: none;
      padding: 10px 20px;
      border-radius: 5px;
      font-size: 24px;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    #content button:hover {
      background-color: #fff;
      color: #333;
    }

    #footer {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100px;
      background-color: #333;
      color: #fff;
    }

    #footer p {
      font-size: 24px;
      margin: 0;
    }
  </style>
</head>
<body>
  <header id="header">
    <h1>Welcome to MS Build LAB160</h1>
    <button onclick="showMessage()">Register</button>
  </header>

  <div id="content">
    <h2>LAB160 DevSecOps: How to simplify and integrate security in the SDLC</h2>
    <p>DevSecOps culture requires an important shift in mindset. Not only do you need to prevent breaches through threat models, code reviews, security testing and security development lifecycle (SDL), but assume breaches as well. Perform war game exercises, penetration tests and continuous monitoring. Today, we cover Defender for DevOps which helps unify, strengthen and manage multi-pipeline DevOps security. </p>
    <button onclick="showMessage()">Learn more</button>
  </div>

  <footer id="footer">
    <p>&copy; 2023 MS Build LAB160</p>
  </footer>

  <script>
    function showMessage() {
      alert("Thanks for attending LAB160! Let's secure the world one code at a time!");
    }
    const aws_secret_access_key="CWaiyZjNC212f9P7hcxG17Ae803jEdFu8pzfryqf";
  </script>
</body>
</html>
```

21. **Commit** the file and click **Create a Pull Request**. Enter a title and hit **Create**.
22. Once the build validation pipeline completes, wait 1-2 minutes and view the Pull Request under **Repos>Pull Requests**.
23. Read the annotations and remediation guidance provided in the PR. Go back to your files, hit **edit**, and fix the issues relating to **TLS, HTTPS**, and **secrets** using the remediation guidance in the Pull Request. Marked the **managed identity** annotation as **won't fix**.
24. Click **Commit** and merge the Pull Request to the main branch with the new changes by clicking **Complete**. The issues you fixed should now be tagged as **Resolved**. You can select to delete this branch after merging.

## Exercise 2: Deploy a healthy Web Application and a website##
**Objective**: Deploy your healthy web application and website to production.

1. Create a new branch by opening the dropdown menu **main**, and click + **New Branch**.
2. Call your branch **BranchApp1**, and click **Create**.
3. Create a new file in the newly created branch, called **webApp.yml**.
4. Copy the following code in your newly created file. Change {YOUR_SERVICE_CONNECTION_NAME} to the service connection you created in exercise 1. Change {YOUR_RESOURCE_GROUP_NAME} to 'RG1':

json
```
trigger:
- main

pool:
  vmImage: 'windows-latest'

steps:
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: '{YOUR_SERVICE_CONNECTION_NAME}'
    subscriptionId: '3259a221-fcbc-4721-8af3-401db3954049'
    action: 'Create Or Update Resource Group'
    resourceGroupName: '{YOUR_RESOURCE_GROUP_NAME}'
    location: 'West US'
    templateLocation: 'Linked artifact'
    csmFile: 'ContosoFinanceAppProd.json'
    deploymentMode: 'Incremental'
```

5. Click **Commit** and **Create a pull request**.
6. Set the Pull Request to auto-complete when finished. You can select to delete this branch after merging.
7. Once the merge completes, navigate to **Pipelines** (the 4th icon on the left side menu), and open **Pipelines**.
8. Click on **New pipeline** on the top right of the page, and create a new pipeline that connects to your ADO repository.
9. When configuring your pipeline, select 'Azure Repos Git', 'Repo{YOUR_USER_ID}' for the repository, and ‘Existing Azure Pipelines YAML file’.
10. Select **webApp.yml** and run the pipeline. You may need to authorize the pipeline to use the service connection by opening the Job and granting access.

Now that the pipeline of our application is defined in ADO, we want to deploy our website in our newly created resource.

11. Open the Azure portal in a new tab at https://portal.azure.com/ and navigate to your resource group: **RG1**. Open the **AppServicePlan** and change the pricing plan from **F1** to **B1**. Now, navigate back to the RG open the Web App that we provisioned in the first exercise. Leave this tab open for now, we will come back to this step.
12. Go back to the Azure DevOps project **DevSecOps Lab**.
13. Create a new branch by opening the dropdown menu **main**, and click + **New Branch**.
14. Call your branch **BranchWeb1**, and click **Create**.
15. Create a new file in the newly created branch, called **webDeploy.yml**.
16. Copy the following code in your newly created file. Change {YOUR_SERVICE_CONNECTION_NAME} to the service connection you created in exercise 1. Change {YOUR_WEB_APP_NAME} to the name of the App Service in the resource group you opened in step 11:

json
```
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.x'
    addToPath: true

- task: ArchiveFiles@2
  displayName: 'Create ZIP file'
  inputs:
    rootFolderOrFile: 'MyWebApp.html'
    includeRootFolder: false
    archiveType: 'zip'
    archiveFile: 'htmlFiles.zip'
    replaceExistingArchive: true

- task: AzureWebApp@1
  displayName: 'Azure Web App Deploy'
  inputs:
    azureSubscription: '{YOUR_SERVICE_CONNECTION}'
    appName: '{YOUR_WEB_APP_NAME}'
    package: 'htmlFiles.zip'
    deploymentMethod: 'auto'
    appType: webApp
```

17. Click **Commit** and **Create a pull request**.
18. Set the Pull Request to auto-complete when finished.
19. Once the merge completes, navigate to **Pipelines** (the 4th icon on the left side menu), and open **Pipelines**.
20. Click on **New pipeline** on the top right of the page, and create a new pipeline that connects to your ADO repository.
21. When configuring your pipeline, select 'Azure Repos Git', 'Repo1' for the repository, and ‘Existing Azure Pipelines YAML file’.
22. Select **webDeploy.yml** and run the pipeline. You may need to authorize the pipeline to use the service connection by opening the Job and granting access.
23. Open the Azure portal using your credentials and make sure that the resource is running.
24. In the **Overview** tab, click on **Browse**.
25. You should see an error saying "You do not have permission to view this directory or page."

**BONUS**: find a way to see your website deployed :)

If you can see your website, you have successfully completed the **Lab 160**.

Thank you!


