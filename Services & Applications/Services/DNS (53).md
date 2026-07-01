
```python
> nslookup -query=mx '<Domain>' -server '<DNS-IP>'
> nslookup -query=ns '<Domain>'  -server '<DNS-IP>'
> nslookup -query=any '<Domain>' -server '<DNS-IP>'
> dig '<Domain>'
> dig '<Domain>' A
> dig '<Domain>' AAAA
> dig '<Domain>' PTR
> dig '<Domain>' NS
> dig '<Domain>' MX

> nmap --script dns-brute --script-args dns-brute.threads=12 '<Domain>'

> fierce -dns '<Domain>'
> fierce -dns '<Domain>' -dnsserver '<DNS>'

> dnsenum --dnsserver 10.129.172.166 --enum -p 0 -s 0 -o subdomains.txt -f subdomains-top1million-110000.txt inlanefreight.htb

> dig ns inlanefreight.htb @10.129.14.128
  ### hitting known name servers
  
> dig CH TXT version.bind 10.129.120.85
  ### querying DNS server's version
  
> dig any inlanefreight.htb @10.129.14.128 
  ### show all dns enteries available to be disclosed
```

### Resolve DNS IP to Domain name.

```python
dig '@172.16.5.10' -x '172.16.5.10' +nocookie
```

---

## Brute force

```python
fierce --domain '<Domain>' --range <Range> --dns-servers '<IP>' --subdomain-file '<wordlist>'
```

### Brute force with Bash

```python
for name in $(cat /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt); do host $name.sportsfoo.com '172.16.5.10' -W 2; done | grep 'has address'
```

---

## AXFR Zone Transfer

```python
dig axfr @ns1.inlanefreight.htb inlanefreight.htb

dig axfr @10.129.110.213 inlanefreight.htb
```

---

## Subdomain Enumeration

```shell
./subfinder -d inlanefreight.com -v     
```

## Subbrute

```shell
echo "ns1.inlanefreight.com" > ./resolvers.txt

./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
```

---
