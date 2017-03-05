# 2.2.b.ii Cloud Provider Option

For the cloud provider I have chosen Azure as it has a lot of resources for .NET applications. To allow the solution to be easily integrated in a CI/CDD environment my choice has been Azure PowerShell cmdlets for ARM that can be installed executing

```
# Install the Azure Resource Manager modules from the PowerShell Gallery
Install-Module AzureRM
```

>Azure Resource Manager (ARM) is a service that allows to manage all the resources that you can create in Azure. JSON files can be used to describe the required infrastructure. [basic instructions](https://docs.microsoft.com/en-us/powershell/azureps-cmdlets-docs/)

The following are the commands that need to be executed to connect to an Azure account and that have to be integrated in a powershell script that can be triggered once the build process is complete from the jenkins build.

```powershell
Login-AzureRmAccount
```

After that we can ask to deploy the whole template. Notice that location of the template can vary depending on the real environment that it will be executed.

```powershell
$BasePath = (Get-Item -Path ".\" -Verbose).FullName
$SecurePwd = ConvertTo-SecureString "passw.1234" -AsPlainText -Force
New-AzureRmResourceGroupDeployment -Name ASGApp -ResourceGroupName TestASG -TemplateUri $BasePath\_azuredeploy.json -envPrefixName asg -username "asgwebsite" -password $SecurePwd -webSrvVMSize "Standard_DS2" -numberOfWebSrvs "2" -sqlVMSize "Standard_DS2" -storageAccountType "Premium_LRS" -database ""
```

This will be the output received confirming that the infrastructure has been created

```
DeploymentName          : ASGApp
ResourceGroupName       : TestASG
ProvisioningState       : Succeeded
Timestamp               : 03/03/2017 23:51:59
Mode                    : Incremental
TemplateLink            :
Parameters              :
                          Name             Type                       Value
                          ===============  =========================  ==========
                          envPrefixName    String                     asg
                          username         String                     asgwebsite
                          password         SecureString
                          webSrvVMSize     String                     Standard_DS2
                          numberOfWebSrvs  Int                        2
                          sqlVMSize        String                     Standard_DS2
                          storageAccountType  String                     Premium_LRS

Outputs                 :
DeploymentDebugLogLevel :
```

And in Azure control panel we can confirm that the machines have been created and are running.

![azure panel](resources/22bii-Virtual machines - Microsoft Azure.png){:class="img-responsive"}

## Template Details

I have started from a [template](https://azure.microsoft.com/en-us/resources/templates/iis-2vm-sql-1vm/) that creates two Windows Server 2012R2 VM(s) with IIS configured using DSC. It also installs one SQL Server 2014 standard edition VM, a VNET with two subnets, NSG, load balancer, NATing and probing rules. One of the best things about using modern infrastructure providers is the amount of resources available to start a project from a sample.

The template includes some custom sections to execute powershell scripts to deploy the web application and modify the configuration variables. 

```json
"commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -File deployWebApp.ps1 -sqlserver ',variables('sqlPublicIP'))]"
```

The whole azure deploy json template can be found in [azure/azuredeploy.json](azure/azuredeploy.json) folder.

A proposal for the deployment script will be something like

```powershell
<#
    .SYNOPSIS
        Downloads and configures web app
#>

Param (
    [parameter(mandatory=$true)]
    [string]$sqlserver
)

# Folders
New-Item -ItemType Directory c:\temp
New-Item -ItemType Directory c:\webApp

# Download app
Invoke-WebRequest  https://myArtifacts/app.zip -OutFile c:\temp\app.zip
Expand-Archive C:\temp\app.zip c:\webApp

# Replace web config
$webConfig = 'C:\webApp\Web.config'
$doc = (Get-Content $webConfig) -as [Xml]
$root = $doc.get_DocumentElement();
$newCon = $root.connectionStrings.add.connectionString.Replace('Data Source=.\','Data Source=$sqlserver');
$root.connectionStrings.add.connectionString = $newCon
$doc.Save($webConfig)

# Configure iis
Remove-WebSite -Name "Default Web Site"
Set-ItemProperty IIS:\AppPools\DefaultAppPool\ managedRuntimeVersion ""
New-Website -Name "ASGWebApp" -Port 80 -PhysicalPath C:\webApp\ -ApplicationPool DefaultAppPool
& iisreset

```

This script will be copied at provisiong time to the azure web instances.

### Resources
The following resources are created by this template:
- 1 or 2 Windows 2012R2 IIS Web Servers.
- 1 SQL Server 2014 running on premium or standard storage.
- 1 virtual network with 2 subnets with NSG rules.
- 1 storage account for the VHD files.
- 1 Availability Set for IIS servers.
- 1 Load balancer with NATing rules.

<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/iis-2vm-sql-1vm/images/resources.png" />

## Architecture Diagram
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/iis-2vm-sql-1vm/images/architecture.png" />

