# -DHCP
These are some of the Scrips related to Configuring DHCP Server in Windows Server 2019

!! note that these are some examples and you can costomise them based on your network

___

## Assigning an static IP address to the DHCP server

    New-NetIPAddress -IPAddress 10.0.0.3 -InterfaceAlias "Ethernet" -DefaultGateway 10.0.0.1 -AddressFamily IPv4 -PrefixLength 24 
    Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.0.0.2


## Rename Computer (Optional)
  
    Rename-Computer  "DHCP1"
    Restart-Computer

## Join the computer to the domain

 	Add-Computer -DomainName test.com -Restart

## Install DHCP

	Install-WindowsFeature -Name DHCP -IncludeManagementTools

## Create DHCP security groups

	netsh dhcp add securitygroups
	Restart-Service dhcpserver

## Authorize the DHCP server in Active Directory (Optional)

	Add-DhcpServerInDC -DnsName DHCP1.test.com -IPAddress 10.0.0.3

 
(To verify that the DHCP server is authorized in Active Directory:)

    Get-DhcpServerInDC 
    

## Set server level DNS dynamic update configuration settings (Optional)

    Set-DhcpServerv4DnsSetting -ComputerName "DHCP1.test.com" -DynamicUpdates "Always" -DeleteDnsRRonLeaseExpiry $True
    
( You can use the following command to configure the credentials that the DHCP server uses to register or unregister client records on a DNS server: )
    
    $Credential = Get-Credential
    Set-DhcpServerDnsCredential -Credential $Credential -ComputerName "DHCP1.corp.contoso.com"

## Configure the Scope

    Add-DhcpServerv4Scope -name "Corpnet" -StartRange 10.0.0.1 -EndRange 10.0.0.254 -SubnetMask 255.255.255.0 -State Active
    
    Add-DhcpServerv4ExclusionRange -ScopeID 10.0.0.0 -StartRange 10.0.0.1 -EndRange 10.0.0.15
    
    Set-DhcpServerv4OptionValue -OptionID 3 -Value 10.0.0.1 -ScopeID 10.0.0.0 -ComputerName DHCP1.test.com
    
    Set-DhcpServerv4OptionValue -DnsDomain test.com -DnsServer 10.0.0.2

## Get a List Of Scops

	  Get-DhcpServerV4Scope

## How Client Receive IP From DHCP (?)

  	Ipconfig / Release
  	Ipconfig /ReNew
  	Ipconfig \All

# -DHCP Failover
---
## Create an active-passive failover relationship 
This example creates a hot standby, or active-passive, failover relationship between the DHCP server services that runs on the computers named dhcpserver.contoso.com and dhcpserver2.contoso.com with the scopes 10.10.10.0 and 10.20.20.0 present on the DHCP server service that runs on the computer named dhcpserver.contoso.com added to the failover relationship. These scopes will be created on the partner DHCP server service that runs on the computer named dhcpserver2.contoso.com as part of the failover relationship creation. The DHCP server service that runs on the computer named dhcpserver.contoso.com will be the standby DHCP server service and the DHCP server service that runs on the computer named dhcpserver2.contoso.com will be the active DHCP server service in the failover relationship.
	Add-DhcpServerv4Failover -ComputerName "dhcpserver.contoso.com" -Name "SFO-SIN-Failover" -PartnerServer "dhcpserver2.contoso.com" -ServerRole Standby -ScopeId 10.10.10.0,10.20.20.0

  ## reate an active-active failover relationship with specified load balance amount 
	Add-DhcpServerv4Failover -ComputerName "dhcpserver.contoso.com" -Name "SFO-SIN-Failover" -PartnerServer "dhcpserver2.contoso.com" -ScopeId 10.10.10.0,10.20.20.0 -LoadBalancePercent 70 -MaxClientLeadTime 2:00:00 AutoStateTransition $True -StateSwitchInterval 2:00:00

 # -ADDC
 ---
 ## Install ADDS 
 	install-windowsfeature AD-Domain-Services -IncludeAllFeature -IncludeManegmentTools 
	                                
**Add-ADDSReadOnlyDomainControllerAccount**--> Install read only domain controller

## Promote Server as DC 
	Get-command install-ad

**Install-ADDSDomain**--> Install first domain controller in a child or tree domain

**Install-ADDSDomainController**--> Install additional domain controller in domain

**Install-ADDSForest**--> Install first domain controller in new forest

**Test-ADDSDomainControllerInstallation**--> Verify prerequisites to install additional domain controller in domain

**Test-ADDSDomainControllerUninstallation**--> Uninstall AD services from server

**Test-ADDSDomainInstallation**--> Verify prerequisites to install first domain controller in a child or tree domain

**Test-ADDSForestInstallation**--> Install first domain controller in new forest

**Test-ADDSReadOnlyDomainControllAccountCreation**-->Verify prerequisites to install read only domain controller

**Uninstall-ADDSDomainController**--> Uninstall the domain controller from server

## Config ADDS 

	Get -help install-ADDSDomainControler (to see the Switches)

	Install-ADDSForest  -CreateDnsDelegation:$false   -DatabasePath “C:\Windows\NTDS”
	-DomainMode “Win2019”   -DomainName “test.com”
	-DomainNetbiosName “TEST.COM”   -ForestMode “Win2016”   -InstallDns:$true
	-LogPath “C:\Windows\NTDS”   -NoRebootOnCompletion:$false
	-SysvolPath “C:\Windows\SYSVOL”  -Force:$true

# -RAID-5
---

	DiskPart
	List Disk 
 
**(if any of the disk that you needs are offline, tune it to online by selecting it:)**

	Select Disk 3
	Online Disk
 
 And then Continue:
 
	Attribite Disk Clear Readonly
	Convert Dynamic
	List Disk
	List Volume 
	Select Volume 3
	Creat Volume Raide Disk= 3,4,5
	Assighn Letter=V
	Exit
	Format V: /q /FS:NTFS
	Y
	RAID-5
	DiskPart
	List Volume

# -Creat & Share Folder

---

## Creat Folder

	New-Item -Path "C:\" -Name "Temp" -ItemType Directory
 
## Verify Existing 

	if (Test-Path -Path C:\Path\To\Folder)   {
    		Write-Host "Folder already exists."} 
      	else {
		Write-Host "Folder does not exist."   }

## Sharing Folder

	Powershell
	New-SmbShare -Name NomDuPartage -Path " V:\WSC\" -FullAccess "Contoso\Administrator"

## Verify Sharing

	Get-SmbShare
	Access the UNC path (\\Server\Shared)

## Create an encrypted SMB share

	New-SmbShare -Name "Data" -Path "J:\Data" -EncryptData $True

## Create an SMB share with Multiple Permissions

	$Parameters = @{{
    		Name = 'VMSFiles'
    		Path = 'C:\ClusterStorage\Volume1\VMFiles'
    		ChangeAccess = 'CONTOSO\Finance Users','CONTOSO\HR Users'
    		FullAccess = 'Administrators'
	}
	New-SmbShare @Parameters

# -File Server

---

## Create a hard limit quota

	New-FsrmQuota -Path "C:\Shares" -Description "limit usage to 1 GB." -Size 1GB

## Change the size limit of a quota
	Set-FsrmQuota -Path "C:\Shares" -Description "limit usage to 1.5 GB" -Size 1.5GB

## Create a quota based on a quota template

	New-FsrmQuota -Path "C:\Shares" -Description "limit usage to 100 MB based on template." -Template "100 MB Limit"

## Change the size limit of all quotas that are derived from a quota template

	Set-FsrmQuotaTemplate "1GB limit" -Description "limit usage to 1.5 GB." -Size 1.5GB -UpdateDerived

## Create a file group

This command creates a file group named "Non-HTML text files". The command indicates that files that end in txt or ml are included in the file group, and that files that end in .html are not included in the file group.

	New-FsrmFileGroup -Name "Non-HTML text files" -IncludePattern @("*.txt", "*ml") -ExcludePattern "*.html"

## Create a file screen exception

	New-FsrmFileScreenException -Path "C:\Share1-IncludeGroup" -IncludeGroup " Non-HTML text files "

