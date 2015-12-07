#Windows Server 2012 Core Deployment met PowerShell

##Voorbereiding
PowerShell als default shell instellen
```Powershell
PowerShell.exe PS > Set ItemProperty "HKLM:\Software\Microsoft\Windows NT\ CurrentVersion\winlogon" Shell PowerShell.exe
```

Guest additions installeren
```Powershell
.\VBoxGuestAdditions
```


##Basisinstellingen
Het systeem een naam gegeven (Assv1)
```Powershell
Rename –Computer –NewName Assv1
Restart-Computer
```

Tijdzone veranderd
```Powershell
TZutil /s “Romance Standard Time”
```

Netwerkcontrollers instellen
```Powershell
New-NetIpAddress –IPAddress 192.168.210.11 –InterfaceAlias Ethernet –DefaultGateway 192.168.210.1 –AddressFamily Ipv4 –PrefixLength 24
```


##Serverinstellingen

DHCP inschakelen
```Powershell
Set-NetIpInterface –InterfaceAlias Ethernet –Dhcp enabled
Install-WindowsFeature DHCP  –IncludeAllSubFeature  –IncludeManagementTools 
```

DNS Client settings
```Powershell
Set-DnsClientServerAddress –InterfaceAlias Ethernet –ServerAddresses 192.168.110.1
```

Active Directory configureren
```Powershell
Install-WindowsFeature AD-Domain-services  –IncludeAllSubFeature  –IncludeManagementTools
```

Remote Server administrator tools
```Powershell
Install-WindowsFeature RSAT  –IncludeAllSubFeature  –IncludeManagementTools
```

Service voor shares te kunnen maken
```Powershell
Install-WindowsFeature FileandStorage-services –IncludeAllSubFeature  –IncludeManagementTools
```

.Net Framework installeren
```Powershell
Install-WindowsFeature NET-Framework-45-features  –IncludeAllSubFeature  –IncludeManagementTools
```

Active Directory Domain Service rol toevoegen
```Powershell
Install‑ADDSForest ‑DomainName Assengraaf.nl ‑SafeModeAdministratorPassword (ConvertTo‑SecureString Test123 ‑AsPlainText ‑Force) -DomainMode Win2012 ‑DomainNetbiosname Assengraaf ‑ForestMode Win2012 ‑InstallDNS
```

