
## Gaining Situational Awareness

Typically we'll want to run a few basic commands to orient ourselves:
`whoami`, `id`, `hostname`, `ip a`, `sudo -l`.

```sh
> cat /etc/os-release ### os and its version

> echo $PATH ### $PATH

> env ### environment variables

> lscpu ### CPU type/version

> cat /etc/shells ### available shells on the server

> lpstat ### information about any attached printers

> lsblk ### info about attached drives un/mounted 

> cat /etc/fstab ### may contain creds

> netstat -rn   # -------
                #       | -------->  routing table 
> route         #--------

> cat /etc/resolv.conf ### checking for internal DNS

> arp -a ### check hosts we can comminute to 

> df -h ### mounted FS

> cat /etc/passwd ### (may contain hashes occassionally)
  root:x:0:0:root:/root:/bin/bash
# user:pass:UID:GID:UID-info:Home-dir:shell

> cat /etc/passwd | cut -f1 -d: ### gives only users

> grep "sh$" /etc/passwd ### login shells

> cat /etc/groups ### check for groups existing on a machine

> getent group sudo ### lists members of a specfific group 
 
> ls /home ### check for any readbale files/confs in this from any user 

> cat /etc/fstab | grep -v "#" | column -t ### unmounted FS 

> find / -type f -name ".*" exec ls -l {} \; 2>/dev/null | grep x1o3 
  ### hidden files

> find / -type d -name ".*" -ls 2>/dev/null ### hidden dirs
```

`/tmp` and `/var/tmp`  stores data for `30` days but `/tmp` only for `10`. `/tmp` is wiped at every restart so `/var/tmp` is used by files which need to retain data. `/dev/shm` is also a temp dir. 

We should also check to see if any defences are in place enumerate them:

- [Exec Shield](https://en.wikipedia.org/wiki/Exec_Shield)
- [iptables](https://linux.die.net/man/8/iptables)
- [AppArmor](https://apparmor.net/)
- [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
- [Fail2ban](https://github.com/fail2ban/fail2ban)
- [Snort](https://www.snort.org/faq/what-is-snort)
- [Uncomplicated Firewall (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)

---
## Services and Internals

```sh
ip a ### current networking of machine

cat /etc/hosts ### show hosts we can communicate to 

lastlog ### shows the last login time of users

w/who/finger ### these 3 commands to show who is login with us

find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null ### finding any kind of history files

ls -la /etc/cron.daily/ ### cron jobs 

find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n" ### proc

apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list ### show all the installed pacakges

ls -l /bin /usr/bin/ /usr/sbin/ ### check for compiled binaries

sudo -v ### checking for legacy sudo versions (same for other binaries)

for i in $(curl -s https://gtfobins.org/api.json | jq -r '.executables | keys[]'); do if grep -q "$i" installed_pkgs.list; then echo "Check for GTFO: $i";fi; done ### compare existing binaries with vulnerable GTFOBins versions

strace ping -c1 10.129.112.20 ### trace system calls 

find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null ### configs

find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share" 
### scripts

ps aux | grep root ### check runnig scripts
```

---
