# Infrastructure as Code - ARM Templates

Azure Resource Manager (ARM) is the native platform for infrastructure as code (IaC) in Azure. It enables the user to centralize the management, deployment, and security of Azure resources through Json-syntax. ARM groups resources into containers that group Azure assets together. Therefore ARM can be used to deploy assets from multiple Azure resource provider services. The advantage lies in using reusable templates which can be orchestrated and versioned. 

This reporsiotory consist of multiple ARM templates that are used to deploy the Phoenix Azure Data Platform. 

## Table of Content

- [Infrastructure as Code - ARM Templates](#infrastructure-as-code---arm-templates)
  * [Features](#features)
  * [Technology](#technology)
  * [Installation](#installation)
  * [Development](#development)
  * [Naming Convention](#namingconvention)
  * [Deployment](#deployment)
  * [ARM Template Parameters](#ArmTemplateParameters)

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
## Naming Convention
The Azure Services have the following naming convention:
```
<countryCode>-dp-<dataProduct><environment>-<azure Service abbreviation>
```
countryCode = abbreviation of the country, e.g. germany = de, or something general europe = eu
dataProduct = abbreviation of the data product that is defined
environment = development environment, e.g. dev = d
azure Service abbreviation = Azure Data Factory = adf, Azure Data Lake = dls and so on
## Deployment
The following Azure services will be deployed by default and these can not be changed without adapting the main.json. It is recommended to have a 2 step deployment:

1. Deployment of the **shared resource group**
2. Deployment of the **data product resource group**

The following deployed services are expected:

- **Shared Resource group** 
&nbsp; -Azure VM
&nbsp; -Azure Data Factory with self-hosted integration runtime
&nbsp; -Azure Key Vault
&nbsp; -Storage Account
&nbsp; -Automated installation of self-hosted integration runtime with registration to Azure Data Factory 
&nbsp; -Automated installtion of OpenJDK with register of JAVA_HOME system variable for parquet support
- **Data product resource group**
&nbsp;- Azure Data Factory with linked Services to the Azure services
&nbsp;- Azure Synapse Analytics without dedicated SQL Pool and Spark Pool
&nbsp;- Azure Data Lake
&nbsp;- Azure SQL Db
&nbsp;- Azure Databricks
&nbsp;- Azure Key Vault
&nbsp;- Azure Machine Learning
&nbsp;- Application Insights
&nbsp;- Azure Container Registry
&nbsp;- Role Assignment to the relevant services with the managed identities
&nbsp;- Role assignment of the User Object IDs to the relevant Azure services:
 &nbsp;&nbsp;&nbsp;&nbsp;- The object ID can be changed in the **folder = arm-modules/roleAssignment/azuredeploy.json; Parameter = userObjectID**
 &nbsp;- Role assignment of used service principal for the deployment. Can be used for Azure Databricks authentication to the Azure Data Lake.
 &nbsp;&nbsp;&nbsp;&nbsp;-The service principal object ID can be changed in the **folder = arm-modules/roleAssignment/azuredeploy.json; Parameter = spObjectID**

#### Deployment - Optional Services
The following services are optional for additional deployment on top of the services mentioned in the **Data product resource group**. These can be enabled or disabled in the next section **ARM Template Parametrs**
- **Data product resource group**
&nbsp;- Azure API Management
&nbsp;- Cosmos Db
&nbsp;- Eventhub
&nbsp;- Azure Function
&nbsp;- Azure Iothub
&nbsp;- Azure Logic App
&nbsp;- Azure Stream Analytics

## ARM Template Parameters
The following paarmeters can be used to give specific input for the deployment and decide the naming of the services and what to optionally deploy or not.

- **environment** (data type = string) : 
Sets the development environment and is used in the naming convention of the services (d = dev, t = test, p = prod).
- **dataProduct** (data type = string) :
Abbreviation of the data product. Is used in the naming convention of the services .
- **dataProductSize** (data type = string) : 
Tshirt-Size for the services. It is used to determine the compute size of the specific services (s = small, m = medium, l = large).
- **adbDeployment** (data type = boolean) : 
Determines to deploy or not to deploy the Azure Databricks related vnet, delegated subnets and the network security group. Mandatory to set abdDeployment = true in initial deployment. In case of redeployment it is necessary to set adbDeplyoment = false.
- **apiManagementDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Azure API Management
- **cosmoDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Cosmos Db
- **eventhubDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Eventhub
- **functionDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Azure Function
- **iotHubDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Azure Iothub
- **logicAppDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Azure Logic App
- **streamAnalyticsDeployment** (data type = boolean) :
Determines to deploy or not to deploy the optional service Azure Stream Analytics