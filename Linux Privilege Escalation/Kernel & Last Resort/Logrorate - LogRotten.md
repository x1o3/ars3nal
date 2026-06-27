
To prevent the hard disk from overflowing, `logrotate` takes care of archiving or disposing of old logs. This tool is usually started periodically via `cron` and configured in file `/etc/logrotate.conf`. 

---
## Exploit: [logrotten](https://github.com/whotwagner/logrotten).

To exploit `logrotate`, we need some requirements that we have to fulfill.

1. We need `write` permissions on the log files
2. Logrotate must run as a privileged user or `root`
3. vulnerable versions:
    - 3.8.6, 3.1.1.0, 3.15.0, 3.18.0

Before running the exploit, we need to determine which option `logrotate` uses in `logrotate.conf`.

```sh
grep "create\|compress" /etc/logrotate.conf | grep -v "#"
	create ### we have to use the exploit adapted to this function.

./logrotten -p ./payload /tmp/tmp.log
```

---