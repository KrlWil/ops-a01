##Running commands

In dit hoofdstuk gaan we niet programmeren of scripten, maar commands uitvoeren.
PowerShell is zeer vergelijkbaar met Cmd.exe, de standaard Command Line Interface van Windows systemen, en Unix shells, zoals Bash, maar veel moderner.
Dus qua interface komt het sowieso bekend voor.

###De structuur van een command

De onderstaande afbeelding toont de basis structuur van een complexer PowerShell command.

![structuur](https://i.gyazo.com/16e542ced76bd1ea45bbd0315c3ea193.png)

Om het wat duidelijker te maken zullen we de onderdelen meer gedetaileerd bekijken:
* De cmdlet die hier wordt gebruikt is Get-Eventlog, op cmdlets komen we later terug.
* De eerste parameter is -LogName, met als waarde Security (de waarde bevat geen spaties of punten, dus moet het niet tussen aanhalingstekens)
* De tweede parameter is -Computername, deze heeft 2 waarden, gescheiden door een komma
* De derde parameter is -Verbose, dit is een switch-parameter, dat wil zeggen dat het geen waarde nodig heeft

Parameters starten altijd met een '-' en er is geen spatie tussen de '-'en de parameter.
Niets is ook hoofdlettergevoelig.

###Commands vergemakkelijken

PowerShell heeft ook afkortingen voor verscheidene cmdlets.
Om deze te vinden gebruiken we het commando "get-alias"
Voorbeeld:
```Powershell
PS C:\> get-alias -Definition "Get-Service"
Capability Name
---------- ----
Cmdlet gsv -> Get-Service
```
Hier zien we dat we "gsv" kunnen gebruiken in plaats van "get-service".

PowerShell verplicht u bovendien niet om uw commands volledig uit te typen.
Bijvoorbeeld kan men gewoon "-comp" gebruiken in plaats van de volledige parameter "-computerName".
Er moeten wel genoeg letters staan zodat elke afkorting ook uniek is.
"-com" werkt bijvoorbeeld niet omdat er ook een parameter "-composite" is.

###Show-command

Als men problemen heeft met de syntax van een command, kan men de cmdlet "Show-Command" gebruiken.

![show command](https://i.gyazo.com/0ddaa346c3342605d117e47b0306c017.png)

De Show-Command cmdlet geeft een venster weer met al de parameters die men dan kan invullen van een cmdlet naar keuze.
In de afbeelding hierboven wou ik als logname Security, als computername localhost en daarvan wou ik de laatste 50 entries zien. Als men dan op run klikt maakt PowerShell het onderstaande command: 
```Powershell
PS C:\Users\Robin> Get-EventLog -LogName Security -ComputerName localhost -Newest 50
```







##Errors voorkomen/oplossen

Voor men fouten probeert te verbeteren in een script, moet men het gebruik van het script eerst begrijpen.
Bij de meeste simpele scriptjes zoals bijvoorbeeld Get-Bios.ps1 valt er niet te veel te verbeteren, aangezien er geen inputs van de command line zitten in dit script.

```Powershell
Get-Bios.ps1
Get-WmiObject -class Win32_Bios
```


###Ontbrekende parameters

In het Get-BiosInformation.ps1 script is er een parameter computerName, dat toelaat om een pc te kiezen als doel van het script.
Als het script geen waarde krijgt voor de computerName parameter, faalt het omdat Get-WmiObject een waarde voor de parameter nodig heeft.
Om het probleem op te lossen gaat het script op zoek naar een $computerName variabele.
Als deze variabele ontbreekt gaat het script een waarde proberen geven aan de variabele.
De volgende lijn van het script zorgt hiervoor: 
If(-not($computerName)) { $computerName = $env:computerName }

```Powershell
Get-BiosInformation.ps1
Param(
  [string]$computerName
) #end param

Function Get-BiosInformation($computerName)
{
  Get-WmiObject -class Win32_Bios -computerName $computername
} #end function Get-BiosName

If(-not($computerName)) { $computerName = $env:computerName }
Get-BiosInformation -computerName $computername
```


Men kan ook de parameters verplicht maken zodat de gebruiker een waarde voor de parameter moet voorzien.
Om een parameter verplicht te maken gebruiken we Mandatory als parameter attribuut.

```Powershell
Param(
  [Parameter(Mandatory=$true)]
  [string]$drive,
  [string]$computerName = $env:computerName
) #end param
```

Als men dit script dan gaat runnen gaat powershell een waarde vragen voor de parameter drive.

###Try/Catch/Finally

Bij een Try/Catch/Finally blok, ga je in het Try gedeelte de commands die het script moet uitvoeren zetten.
Als er een error is bij de commands, gaat het script naar het Catch gedeelte, waar de fout wordt opgevangen met een string, om te zeggen dat er een fout gebeurd is.
Hetgeen dat in het Finally gedeelte staat wordt altijd uitgevoerd, ookal was er een error gegenereerd.
In het algemeen zet men daar de "code cleanup", het stukje code dat ervoor gaat zorgen dat de niet gebruikte commands of commands waar er fouten stonden niet worden uitgevoerd.
In het voorbeeldscript staat er een string om te zeggen dat het script beëindigd is.

```Powershell
TestTryCatchFinally.ps1
$obj1 = "Bad.Object"
"Begin test"
Try
  {
    "`tAttempting to create new object $obj1"
    $a = new-object $obj1
    "Members of the $obj1"
    "New object $obj1 created"
    $a | Get-Member
  }
Catch [system.exception]
  {
    "`tcaught a system exception"
  }
Finally
  {
    "end of script"
  }
```


##Server beheren

###Server Manager cmdlets 
PowerShell laat ons toe om features te installeren op onze Windows Server 2012.
Om te werken met de server manager cmdlets moet men eerst de Server Manager Module laden in PowerShell.
Dat doet men via het commando:
```Powershell
Import-Module Servermanager
```
Om deze features te zoeken gebruikt men het "GET-WindowsFeature" cmdlet.
De checkboxes met een X duiden aan welke features reeds geïnstalleerd zijn.
Men kan features installeren of verwijderen met de cmdlets "INSTALL-WindowFeature" en "UNINSTALL-WindowsFeature".
Men kan ook een pipe line gebruiken om meerdere features te installeren.
Het volgende voorbeeld installeert alle features die beginnen met "web-".
```Powershell
Get-WindowsFeature web-* |Install-WindowsFeature
```
Je kan het ook met een komma doen, zoals in het volgende voorbeeld:
```Powershell
Install-WindowsFeature Telnet-Server,Hyper-V
```

###Netwerkbeheer met PowerShell
Om alle interfaces op onze server te bekijken gebruiken we het cmdlet "GET-NetIPInterface". (Men kan de parameter -InterfaceAlias gebruiken om een specifiek interface te gebruiken)
Als we dan een nieuwe interface willen toevoegen, moeten we het cmdlet "NEW-NetIPInterface" gebruiken met de nodige parameters,
zoals de interface alias, het adres, soort van ip-adres en de prefix van het subnet mask.
Hieronder een voorbeeld waarbij ik een interface genaamd Ethernet 2 aanmaak.
```Powershell
New-NetIPInterface -InterfaceAlias "Ethernet 2" -IPAddress 192.168.10.20 `
-AddressFamily IPv4 -PrefixLength 24
```
Men kan ook bindings inschakelen en uitschakelen met Powershell.
Om de bindings van een interface te bekijken gebruikt men:
```Powershell
GET-NetwerkAdapterBinding -InterfaceAlias "Ethernet 2"
```
Om bijvoorbeeld de binding ms_lltdio uit te schakelen gebruikt men het commando:
```Powershell
Disable-NetAdapterBinding -Name "Ethernet 2" -ComponentID ms_lltdio
```
En natuurlijk kan men ook de netwerkadapter zelf uitschakelen door middel van het commando:
```Powershell
Disable-NetAdapter -Name "Ethernet 2"
```


###Group Policy beheer met PowerShell
Voor men Group Policy cmdlets kan gebruiken moet men de Group Policy Management Console installeren, dit doen we zo:
```Powershell
Import-Module ServerManager
```
```Powershell
Add-WindowsFeature GPMC
```
Om al de Commandos binnen deze module te bekijken, gebruiken we:
```Powershell
Get-Command -Module GroupPolicy
```
Om Group Policy Objects (GPO) te bekijken gebruiken we het cmdlet "GET-GPO" waarbij men natuurlijk ook parameters kan gebruiken.
Men kan ook GPO's maken met het cmdlet "NEW-GPO", hieronder een beter voorbeeld: 
```Powershell
New-GPO -Name TestGPO -Comment "This is a GPO for testing purposes"
```


###IIS beheer met PowerShell
Hier ook, om te kunnen werken met de Web Administration cmdlets, moet men de module importeren:
```Powershell
Import-Module WebAdministration
```
Om een website aan te maken gebruiken we het cmdlet "NEW-Website" met de nodige parameters:
```Powershell
New-Website -Name testsite -Port 80 -HostHeader testsite -PhysicalPath
c:\temp
```
Web Binding instellen:
```Powershell
Set-WebBinding -Name 'Default Web Site' -BindingInformation "*:80:"
-PropertyName Port -Value 1234
```
Men kan ook een FTP site maken:
```Powershell
New-WebFtpSite -Name testFtpSite -Port 21 -PhysicalPath c:\test
-HostHeader mySite -IPAddress 127.0.0.1
```
Of een virtuele directory:
```Powershell
New-WebVirtualDirectory -Site "Default Web Site" -Name TestVDir
-PhysicalPath c:\inetpub\virtualdirectory
```
PowerShell kan ook een back-up nemen van de webconfiguratie of die weer herstellen.
Om een back-up te nemen gebruik je "Backup-WebConfiguration" met -Name als parameter.
Als je een back-up wil herstellen gebruik je "Restore-WebConfiguration" met de naam als parameter.


###DNS server beheer met PowerShell
Om een lijst te krijgen van de zones op een DNS server gebruikt men het "GET-DnsServerZone" cmdlet.
Als je de records van een bepaalde zone wilt bekijken gebruikt men het "Get-DnsServerResourceRecord" cmdlet, met de nodige parameter.
```Powershell
Get-DnsServerResourceRecord -ZoneName Test.com
```
Om een resource record toe te voegen gebruikt men het gepaste commando, afgaand van het record type, 
voor een record van type a is het bijvoorbeeld:
```Powershell
Add-DnsServerResourceRecordA -IPv4Address 192.168.2.1 -Name gateway
-ZoneName Test.com
```


