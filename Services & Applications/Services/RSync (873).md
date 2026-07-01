
## Footprinting

```sh
### checking services.
nmap -sV 127.0.0.1 -p 873

### probing for accessible shares.
nc -nv 127.0.0.1 873

### enumerating an open share called dev.
rsync -av --list-only rsync://127.0.0.1/dev

### pulling/syncing files over `-e ssh` is only if ssh is enabled.
rsync -av rsync://127.0.0.1/dev -e ssh
```

This [guide](https://phoenixnap.com/kb/how-to-rsync-over-ssh) is helpful for understanding the syntax for using Rsync over SSH.

---