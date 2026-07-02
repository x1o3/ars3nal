
```powershell
PS C:\htb> Get-MpComputerStatus
### Check Defender

PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
### Check AppLocker

PS C:\htb> $ExecutionContext.SessionState.LanguageMode
### Check if full Language mode or Constrained 
```

---
## LAPS

[Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) is used to randomize and rotate local administrator passwords on Windows hosts and prevent lateral movement.

```powershell
PS C:\htb> Find-LAPSDelegatedGroups
```

The `Find-AdmPwdExtendedRights` checks the rights on each computer with LAPS enabled for any groups with read access and users with "All Extended Rights." 

Users with `"All Extended Rights"` can read LAPS passwords and may be less protected than users in delegated groups, so this is worth checking for.

```powershell
PS C:\htb> Find-AdmPwdExtendedRights
```

The `Get-LAPSComputers` function search for computers that have LAPS enabled when passwords expire, and even the randomized passwords in cleartext if our user has access.

```powershell
PS C:\htb> Get-LAPSComputers
```

---