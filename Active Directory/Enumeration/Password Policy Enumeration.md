
## NULL Session from Windows

```PowerShell
C:\htb> net use \\DC01\ipc$ "" /u:""
```

---

## Password Policies

### Netexec

```shell
nxc smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

### RPC CLient

```shell
> rpcclient -U "" -N 172.16.5.5

  rpcclient $> querydominfo 
  ### Get domain info , servers, users, aliases etc.

  rpcclient $> getdompwinfo 
  ### Get password policies
```

### [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) 

```shell
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

### LDAP

```shell
ldapsearch -H 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

### Net and PowerView

```powershell
C:\htb> net accounts

PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy
```

---