# Exercise 2

## Own Datacenter Option

For this scenario I have chosen VMWare ESXi using the [VMWare PowerCLI cmdlets](https://www.vmware.com/support/developer/PowerCLI/). This technology allows to interact with ESXi servers from the command line which is a good option to automate deployments from a Jenkins or any other CI pipeline.

```powershell
# Connect
Connect-VIServer -Server $ip -User root -Password $password

# Create a win 2012 machine
$machine = "WebServer1"
New-VM -Name $machine -DiskGB 20 -DiskStorageFormat Thin -MemoryGB 2 -CD -GuestId windows9Server64Guest -NumCpu 2

# Copy ISO to machine
$datastore = Get-Datastore "datastore1"
$iso = "en_windows_server_2016_x64_dvd_9718492.iso"
Copy-DatastoreItem D:\ISO\$iso -Destination vmstore:\ha-datacenter\datastore1\$iso
# Mount a CD drive
$vmFile = get-item vmstore:\ha-datacenter\datastore1\$iso
Get-CDDrive -VM $machine | Set-CDDrive -IsoPath $vmFile.DatastoreFullPath –StartConnected $True -Confirm:$False

# Start it
$vmConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec
$vmConfigSpec.BootOptions = New-Object VMware.Vim.VirtualMachineBootOptions
$vmConfigSpec.BootOptions.BootDelay = "5000"
$vm.ReconfigVM_Task($vmConfigSpec)

# TODO add an unattended installation script

Start-VM $machine

```

This should be put in a loop, together with the configuration files for each type of machine we want to create.

### Webserver machine

We can execute a remote powershell script to enable the desired roles like IIS and then deploy the website.

```powershell
# Setup remote session
$ip = (Get-VM -Name $machine).Guest.IPAddress
$pw = convertto-securestring -AsPlainText -Force -String "passw.1234"
$cred = new-object -typename System.Management.Automation.PSCredential -argumentlist "Administrator",$pw

# Copy Website files
$TargetSession = new-pssession -computername $ip -credential $cred
Copy-Item -ToSession $TargetSession -Path $source -Destination "C:\webApp\" -Recurse

# Install IIS
Enter-PSSession -ComputerName $ip -Credential $user
Install-WindowsFeature -Name "Web-Server" -IncludeAllSubFeature -IncludeManagementTools

# Create web site
Remove-WebSite -Name "Default Web Site"
Set-ItemProperty IIS:\AppPools\DefaultAppPool\ managedRuntimeVersion ""
New-Website -Name "ASGWebApp" -Port 80 -PhysicalPath C:\webApp\ -ApplicationPool DefaultAppPool
iisreset
```

### SQLServer

We will need to perform a silent installation of the SQL server instance. SQL server image has to be mounted in the machine and then copy a ConfigurationFile.ini covering all the details that we want to set-up. We will need to create a remote session as in the web site example but this time to copy the required files and install the SQL server.

```ini
; Microsoft SQL Server Configuration file  
[OPTIONS]  
; Specifies a Setup work flow, like INSTALL, UNINSTALL, or UPGRADE.   
; This is a required parameter.   
ACTION="Install"  
; Specifies features to install, uninstall, or upgrade.   
; The list of top-level features include SQL, AS, RS, IS, and Tools.   
; The SQL feature will install the database engine, replication, and full-text.   
; The Tools feature will install Management Tools, Books online,   
; SQL Server Data Tools, and other shared components.   
FEATURES=SQL,Tools
QUIET="True"
UpdateEnabled="False"
SQLSVCSTARTUPTYPE="Automatic"
```

After we should execute the setup on the remote machine to perform the silent installation of the SQLServer

```powershell
.\setup.exe /SQLSVCPASSWORD="passw.1234" /AGTSVCPASSWORD="passw.1234" /ASSVCPASSWORD="passw.1234" /ISSVCPASSWORD="passw.1234" /RSSVCPASSWORD="passw.1234" /IACCEPTSQLSERVERLICENSETERMS /ConfigurationFile=ConfigurationFile.ini
```

## Notes

Alternatively we could have used template machines to speed up the deploment process. The above procedures could be adapted to create a base templates machines so that those could be recreated from scratch when needed. With machines with the OS installed and the IIS or SQL Server pre-installed to whole deployment will be much faster.

I have chosen here the standard powershell tools but other tools for configuration management like Chef will enable many other possibilities to configure particularities on each machine.