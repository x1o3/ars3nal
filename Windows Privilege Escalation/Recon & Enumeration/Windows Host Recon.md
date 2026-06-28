
## Table of Contents:

- [[#Network Information]]
- [[#Enumerating Protections]]
- [[#System Information]]
- [[#User & Group Information]]
- [[#User & Computer Description Field]]
- [[#Named Pipes]]
- [[#Installed Applications & Programs]]

---

## Network Information

```powershell
ipconfig /all ### shows all virtual or physical networks 
arp -a ### hosts we recently ocmmunicated with, ARP cache
route print ### 
```

---

## Enumerating Protections

```powershell
Get-MpComputerStatus 
### Check Windows Defender Status

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections 
### List applocker rules

Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
### Test Applocker policy
```

---

## System Information

```powershell
C:\htb> tasklist /svc 
### shows currently running processes

C:\htb> tasklist | findstr 2384
### find process from PID

C:\htb> set
### maps out all variables including PATH, HOMEDRIVE etc

C:\htb> systeminfo
### patch info/ vm info/ 

C:\htb> wmic qfe
### display HotFixes

PS C:\htb> Get-HotFix | ft -AutoSize
### display HotFixes

C:\htb> wmic product get name
### display installed softwares

PS C:\htb> Get-WmiObject -Class Win32_Product | select Name, Version
### display installed softwares

PS C:\htb> netstat -ano
### display active ports to check running services
```

---

## User & Group Information

```powershell
C:\htb> query user
### logged in users

C:\htb> echo %USERNAME%
### current user

C:\htb> whoami /priv
### current user privileges

C:\htb> whoami /groups
### group inherited rights

C:\htb> net user
### all users on system

C:\htb> net localgroup
### all groups on system

C:\htb> net localgroup onemoregroupugh
### info about the group /// unlikely contains creds

C:\htb> net accounts
### password policy and other account info
```

---

## User & Computer Description Field

```PowerShell
PS C:\htb> Get-LocalUser
### enumerating local user description field

PS C:\htb> Get-WmiObject -Class Win32_OperatingSystem | select Description
### enumerating computer description field
```

---

## Named Pipes

Pipes are essentially files stored in memory that get cleared out after being read.

```powershell
C:\htb> pipelist.exe /accepteula

PS C:\htb> gci \\.\pipe\

C:\htb> accesschk.exe /accepteula \\.\Pipe\lsass -v
### enumerating permissions 

C:\htb> accesschk.exe /accepteula \Pipe\
### revieewing DACLs of all named pipes
```

---

## Installed Applications & Programs

```powershell
C:\htb> dir "C:\Program Files"
### listing common applications

C:\htb> wmic product get name
### display installed programs

PS C:\htb> get-service | ? {$_.DisplayName -like 'Druva*'}
### enumerating running service

PS C:\htb> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation

PS C:\htb> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation

PS C:\htb> $INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize
### listing installed appliations via PS & reg keys
```

Unusual applications like [[Pillaging#[mRemoteNG](https //mremoteng.org/)|mRemoteNG]] can be exploited.

---