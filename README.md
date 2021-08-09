# Infrastructure as Code - ARM Templates

Azure Resource Manager (ARM) is the native platform for infrastructure as code (IaC) in Azure. It enables the user to centralize the management, deployment, and security of Azure resources through Json-syntax. ARM groups resources into containers that group Azure assets together. Therefore ARM can be used to deploy assets from multiple Azure resource provider services. The advantage lies in using reusable templates which can be orchestrated and versioned. 

This reporsiotory consist of multiple ARM templates that are used to deploy the Phoenix Azure Data Platform. 

## Table of Content

- [Infrastructure as Code - ARM Templates](#infrastructure-as-code---arm-templates)
  * [Features](#features)
  * [Technology](#technology)
  * [Installation](#installation)
  * [Development](#development)

## Features

- Generic ARM templates adapted for the Data Mesh concept
- Automated access assignment between the Azure services; ready to immediatley work with the platform
- Automated network assignment with correspondent Service Endpoints
- Sizing through simple T-Shit sizes
- Reusable for different Data Products and development environments

## Technology

ARM is part of a big cloud system and therefore exclusively used in:

- [Microsoft Azure Cloud](https://azure.microsoft.com/en-us/) - Microsoft own cloud
- [JSON](https://www.json.org/json-en.html) - The syntax that ARM is using to deploy the Azure services

Micrososft offers a number of quick start templates to get a feeling of the capabilities of ARM. These can be found on [Github](https://github.com/Azure/azure-quickstart-templates)

## Installation

To manage and deploy ARM templates Azure Cli or Azure powershell is required. The installation guide can be found below:
- [Azure Cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli)
- [Azure Powershell](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-6.2.1)

Install the packages with administrator rights in the Windows Command Line Interface (cmd) or in the Windows Powershell.

The following tools can ease the development effort:
- [Visual Studio Code](https://code.visualstudio.com/) - It is recommended to install the ARM Template extensions from Microsoft
## Development

Commonly the ARM tenplates are either stored in an Azure Storage Account or any **versioning system**, e.g. Azure Repos, Github repository and so on.

Visual Studio Code can be used to connect to a repository, e.g. Azure Repos to locally write ARM Templates and also commit changes to the Azure repos. It is the same way of working with [Git](https://git-scm.com/). Therefore a URL of the repository is needed to connect to it.

In the users preferred terminal the following commands are always used to deploy the written ARM templates.

**Connect to the Azure Account (Powershell) using a Service Principal:**

```powershell
$User = "<Service Principal Application Id>"
$Password = ConvertTo-SecureString -String "<Service Principal Secret>" -AsPlainText -Force
$cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User, $Password
Connect-AzAccount -Credential $cred -Tenant "<Tenant Id of the Azure Subscription>" -Subscription "<Azure Subscription Id>" -ServicePrincipal
```
**Connect to the Azure Account (Azure Cli) using a Service Principal:**

```sh
$User = "<Service Principal Application Id>"
$Password = ConvertTo-SecureString -String "<Service Principal Secret>" -AsPlainText -Force
az login --service-principal -u $User -p $Password --tenant <Tenant Id of the Azure Subscription>
```

**Connect to the Azure Account (Powershell) using a user account:**

```powershell
Connect-AzAccount
Set-AzContext -Subscription "<Azure Subscription Id>"
```
**Connect to the Azure Account (Azure Cli) using a user account:**

```sh
az login
az account set --subscription <name or id>
```

**Deploy the ARM templates to Azure (Powershell):**

```powershell
$TemplateUri = "SAS URL or Github Raw Content URL"
$TemplateParamterUri(optional) = "<SAS URL or Github Raw Content URL>" 
New-AzResourceGroupDeployment -ResourceGroupName "<Resource group name>" -TemplateUri $TemplateUri -TemplateParameterUri $TemplateParamterUri -Verbose
```

**Deploy the ARM templates to Azure (Azure Cli):**

```sh
az deployment group create --resource-group <resource group name> --name <custom deployment name> --subscription <subscription Id> --template-uri <SAS URL or Github Raw Content URL>  --parameters param1='<exampleValue>' param2='exampleValue'
```