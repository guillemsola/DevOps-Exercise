# 2.2.b.i Own Datacenter Option

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

This should be put in a loop, together with the configuration files for each machine we want to boot up.

## Webserver machine

We can execute a remote powershell script to enable the desired roles like IIS.

```powershell
# TODO
```

## SQLServer

We will need to perform a silent installation of the SQL server instance. SQL server image has to be mounted in the machine and then copy a ConfigurationFile.ini covering all the details that we want to set-up.

```
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

Alternatively we could have used template machines to speed up the deploment process. The above procedures could be adapted to create a base templates machines so that those could be recreated from scratch when needed.