
## Nmap

```bash
nmap -sV -p 21 --open <IP>
### identify hosts running FTP

nmap --script ftp-* -p 21 <IP>
### all nmap scripts

nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
### FTP Bounce attack Detection
```

---

## Manual banner grab

```bash
telnet <IP> 21
nc <IP> 21 
```

---

## Anonymous FTP

```sh
use auxiliary/scanner/ftp/anonymous
### Anonymous check with Metasploit

Credentials: anonymous:''
### Anonymous Login
```

---

## Download all files from FTP

```bash
wget -m ftp://anonymous:anonymous@10.10.10.98
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.98
### -m: mirrors the site
```

---