
I mostly prefer use [[ffuf]] and sometimes [[Gobuster]].

---

## Directory Fuzzing

```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```

## Extension Fuzzing

This finds website backend extension.
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```

## Page Fuzzing

This fuzz directories with appropriate extension.
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```

## Recursive Scanning

We can enable recursive scanning with the `-recursion` flag, and we can specify the depth with the `-recursion-depth` flag.
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```

---

## Subdomains & Vhosts Enumeration 

### Subdomains

A sub-domain is any website underlying another domain. (DNS Level)
```shell
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

### Vhosts Fuzzing

VHost is basically a 'sub-domain' served on the same server and has the same IP, such that a single IP could be serving two or more different websites. (Web server level like nginx config)
```shell
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

```sh
gobuster vhost -u http://<IP> -w <wl> --append-domain -t <threads> -o <file> --append-domain
# `-k` flag ignores SSL/TLS certificate errors.
```

### Filtering

```shell
  -ac auto calibratioon of false positives
  
  -mc   Match HTTP status codes, or "all" for everything. (default: 200,204,301,302,307,401,403)
  -ml   Match amount of lines in response
  -mr   Match regexp
  -ms   Match HTTP response size
  -mw   Match amount of words in response

  -fc   Filter HTTP status codes from response. Comma separated list of codes and ranges
  -fl   Filter by amount of lines in response. Comma separated list of line counts and ranges
  -fr   Filter regexp
  -fs   Filter HTTP response size. Comma separated list of sizes and ranges
  -fw   Filter by amount of words in response. Comma separated list of word counts and ranges
```

---

## Parameter Fuzzing

If we get an access denied on a page, this indicates that there is something that identifies the current user like a cookie. We can try to fuzz the right parameter.
#### GET Request Fuzzing
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```
#### POST Request Fuzzing
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

```Note
In *PHP*, "POST" data "content-type", for fuff param fuzzing, the best option is "application/x-www-form-urlencoded". Apart from that PHP accepts "multipart/form-data" and "application/json".
```

---

## Value Fuzzing

#### Custom wordlist
```shell
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```
#### Value Fuzzing
```shell
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```

---