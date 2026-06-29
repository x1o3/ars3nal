
## Common Files

 These may be found in configuration files (`.conf`, `.config`, `.xml`, etc.), shell scripts, bash history, backup (`.bak`) files, within database files or in text files. 

The `/var` directory contains the `web root` for whatever web server is running.

```shell
> find /home/* -type f -name "*.txt" -o ! -name "*.*" 
  ### searching for notes
  
> grep 'DB_USER\|DB_PASSWORD' wp-config.php
  ### greping for database creds from wordpress config

> ls -la /etc/cron.*/
  ### enumerating cron jobs

> tail -n5 /home/*/.bash*
  ### enumerating bash files (inc history) 

> ls -la .ssh/
  ### Whenever finding SSH keys check the `known_hosts` file to find targets.
  
> find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
  ### looking for configuration files in proc
   
> for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
  ### looking for configuration files
  
> for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
  ### looking for configuration files
  
> for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
  ### searching for databases

> for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
  ### searching for scripts

> for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
  ### searching for interesting keywords in log files
```

---

## Memory and cache

### [Mimipenguin](https://github.com/huntergregal/mimipenguin) (Require root permissions)
 
```shell
python3 mimipenguin.py
```
### [LaZagne](https://github.com/alessandroz/lazagne)

```shell
python3 laZagne.py all
```
### Browser credentials

```shell
ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

```shell
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

<SNIP>
"encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr",
"encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE="
<SNIP>
```

Decrypt the credentials with  [Firefox Decrypt](https://github.com/unode/firefox_decrypt).

---
## Enumerating log files

| **File**              | **Description**                                    |
| --------------------- | -------------------------------------------------- |
| `/var/log/messages`   | Generic system activity logs.                      |
| `/var/log/syslog`     | Generic system activity logs.                      |
| `/var/log/auth.log`   | (Debian) All authentication related logs.          |
| `/var/log/secure`     | (RedHat/CentOS) All authentication related logs.   |
| `/var/log/boot.log`   | Booting information.                               |
| `/var/log/dmesg`      | Hardware and drivers related information and logs. |
| `/var/log/kern.log`   | Kernel related warnings, errors and logs.          |
| `/var/log/faillog`    | Failed login attempts.                             |
| `/var/log/cron`       | Information related to cron jobs.                  |
| `/var/log/mail.log`   | All mail server related logs.                      |
| `/var/log/httpd`      | All Apache related logs.                           |
| `/var/log/mysqld.log` | All MySQL server related logs.                     |

---