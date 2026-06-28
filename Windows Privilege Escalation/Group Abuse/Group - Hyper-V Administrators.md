The [Hyper-V Administrators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators) group has full access to all [Hyper-V features](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines). The virtualized Domain Controllers could easily create a clone of the live Domain Controller and mount the virtual disk offline to obtain the `NTDS.dit `file. 

This [blog](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/) shows how upon deleting a VM, `vmms.exe` attempts to restore the original file permissions on the `.vhdx` file and does so as `NT AUTHORITY\SYSTEM`, without impersonating the user. We can delete the `.vhdx` file and create a native hard link to point this file to a protected SYSTEM file with full permissions.

We can leverage [CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) or [CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841), to gain SYSTEM privileges. 

We can try to take advantage of an application on the server that has installed a service running in the context of SYSTEM, which is startable by unprivileged users.

---
## Exploitation Example

```Note
This vector has been mitigated by the March 2020 Windows security updates, which changed behavior relating to hard links.
```

Firefox installs the `Mozilla Maintenance Service`. We can update [this exploit](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1) (a proof-of-concept for NT hard link) to grant our current user full permissions.

```PowerShell
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
### Target File

C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
### Taking Ownership after running the PowerShell script

C:\htb> sc.exe start MozillaMaintenance
### We can replace this file with a malicious one and start it to gain SYSTEM Command Execution
```

---