### Enumerating the Remote Desktop Users Group

```powershell
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Desktop Users
MemberName   : INLANEFREIGHT\Domain Users
SID          : S-1-5-21-3842939050-3880317879-2865463114-513
IsGroup      : True
IsDomain     : UNKNOWN
```

From the information above, we can see that all Domain Users can RDP to this host.

Good to ask yourself: 
-  Does the Domain Users group have local admin rights or execution rights (such as RDP or WinRM) over one or more hosts? (Check on Bloodhound)

---
