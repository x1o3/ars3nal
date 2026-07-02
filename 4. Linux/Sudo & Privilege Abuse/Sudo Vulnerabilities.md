
```sh
sudo -v ### check sudo version
```

---
## CVE-2021-3156

```sh
sudo cat /etc/sudoers | grep -v "#" | sed -r '/^\s*$/d'
### checking privileges of all users or groups
```

[CVE-2021-3156](https://github.com/blasty/CVE-2021-3156) is based on a heap-based buffer overflow vulnerability.

---
## Sudo Policy Bypass -1

[CVE-2019-14287](https://www.sudo.ws/security/advisories/minus_1_uid/) affected all versions below `1.8.28`, which allowed privileges to escalate even with a simple command. It requires only a single prerequisite. It had to allow a user in the `/etc/sudoers` file to execute a specific command.

 If a negative ID (`-1`) is entered at `sudo`, this results in processing the ID `0`, which only the `root` has. This, therefore, led to the immediate root shell.

```sh
sudo -u#-1 id ### -1 results in processing the ID 0 
```

---

