
## Windows Server
### Enumeration

```PowerShell
PS C:\htb> wmic qfe
### Querying Current Patch Level

PS C:\htb> Set-ExecutionPolicy bypass -Scope process
PS C:\htb> Import-Module .\Sherlock.ps1
PS C:\htb> Find-AllVulns
### Running Sherlock
```

### Exploitation

Considering we have command execution (CE), we can gain a meterpreter shell using `smb_delivery` module and then may need to migrate to a `x64` process.

---

## Windows Desktop Versions

### Enumeration

```PowerShell
C:\htb> systeminfo
```

### Exploitation

```sh
> sudo python2 windows-exploit-suggester.py --update
### Updating Microsoft Vulnerability Database, will save contents to a excel file

> python2 windows-exploit-suggester.py --database 2021-05-13-mssb.xls --systeminfo win7lpe-systeminfo.txt
### Running Windoes Exploit Suggester, take the excel and systeminfo
```

With meterpreter shell we can also use this [local exploit suggester module](https://www.rapid7.com/blog/post/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/).

---