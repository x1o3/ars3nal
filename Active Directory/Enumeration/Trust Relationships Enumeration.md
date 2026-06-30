
## ActiveDirectory

```powershell
PS C:\htb> Import-Module activedirectory
PS C:\htb> Get-ADTrust -Filter *
```

---

## PowerView

```powershell
PS C:\htb> Get-DomainTrust 
### Existing Trusts

PS C:\htb> Get-DomainTrustMapping

PS C:\htb> Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
### Enumerate Users in Child Domain
```

---

## netdom

```PowerShell
C:\htb> netdom query /domain:inlanefreight.local trust
### Query Domain Trusts

C:\htb> netdom query /domain:inlanefreight.local dc
### Query Domain Controllers

C:\htb> netdom query /domain:inlanefreight.local workstation
### Query Workstations and Servers
```

---

## [[BloodHound]]

We can also use BloodHound to visualize these trust relationships by using the `Map Domain Trusts` pre-built query. Here we can easily see that two bidirectional trusts exist.

---