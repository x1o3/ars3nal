
## PowerView - need SID

```powershell
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley

PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} 
```

The `ObjectAceType` may give us a `GUID` value which we can google for the rights.

---
## [[BloodHound]]

In BloodHound, we can set the `wley` user as our starting node, select the `Node Info` tab and scroll down to `Outbound Control Rights`

---
