
Certain applications create cron files in the `/etc/cron.d` directory and may be misconfigured to allow a non-root user to edit them.

```sh
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
### looking for world writable files
```

 [pspy](https://github.com/DominicBreuker/pspy) is used to view running processes without the need for root privileges, It works by scanning [procfs](https://en.wikipedia.org/wiki/Procfs).

```sh
./pspy64 -pf -i 1000
### -i is time in microseconds, -pf prints commands and file systems
```

We should also attempt to append our commands to the `end of the script` to still run properly before executing our reverse shell command.

---