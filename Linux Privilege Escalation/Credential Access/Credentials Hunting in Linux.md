
 These may be found in configuration files (`.conf`, `.config`, `.xml`, etc.), shell scripts, a user's bash history file, backup (`.bak`) files, within database files or even in text files. 

The `/var` directory typically contains the `web root` for whatever web server is running on the host. 

```shell
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null

grep 'DB_USER\|DB_PASSWORD' wp-config.php

ls -la .ssh/
### Whenever finding SSH keys check the `known_hosts` file to find targets.
```

---