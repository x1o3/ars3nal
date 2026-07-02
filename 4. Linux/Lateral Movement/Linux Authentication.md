## Passwd file

```sh
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

| Field                                              | Value               |
| -------------------------------------------------- | ------------------- |
| Username                                           | `htb-student`       |
| Password                                           | `x`                 |
| User ID                                            | `1000`              |
| Group ID                                           | `1000`              |
| [GECOS](https://en.wikipedia.org/wiki/Gecos_field) | `,,,`               |
| Home directory                                     | `/home/htb-student` |
| Default shell                                      | `/bin/bash`         |

Usually, we will find the value `x` in this field, indicating that the passwords are stored in a hashed form within the `/etc/shadow` file. However, it can also be that the `/etc/passwd` file is writeable by mistake. This would allow us to remove the password field for the `root` user entirely.

```sh
head -n 1 /etc/passwd

root::0:0:root:/root:/bin/bash
```

This results in no password prompt when attempting to log in as `root`.

---
## Shadow file

```sh
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

| Field             | Value                              |
| ----------------- | ---------------------------------- |
| Username          | `htb-student`                      |
| Password          | `$y$j9T$3QSBB6CbHEu...SNIP...f8Ms` |
| Last change       | `18955`                            |
| Min age           | `0`                                |
| Max age           | `99999`                            |
| Warning period    | `7`                                |
| Inactivity period | `-`                                |
| Expiration date   | `-`                                |
| Reserved field    | `-`                                |
If the `Password` field contains a character such as `!` or `*`, the user cannot log in using a Unix password. However, other authentication methods such as Kerberos or key-based authentication can still be used.

---
## Opasswd

The PAM library (`pam_unix.so`) can prevent users from reusing old passwords. These previous passwords are stored in the `/etc/security/opasswd` file. 

```shell
> sudo cat /etc/security/opasswd

cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1
```

---
## Cracking Linux Credentials

```sh
> sudo cp /etc/passwd /tmp/passwd.bak
> sudo cp /etc/shadow /tmp/shadow.bak 

> unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes

> hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked

> john --single unshadowed
```

---
