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
