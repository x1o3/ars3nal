
## Attack Surface:

- [ ] `Wappalyzer` browser extension - passive tech detection
- [ ] `BuiltWith` - external only, no requests to target
- [ ] [[#WhatWeb]] CLI - active tech fingerprinting
- [ ] [[Nikto]] - web server tech + known vulnerabilities scan
- [ ] [[Nmap]] web discovery scripts - banner grab + service version
- [ ] [[#Banner Grabbing]]
- [ ] [[#Wafw00f]] - identify WAF before you start poking
- [ ] Check HTTP response headers manually (Server, X-Powered-By, X-Generator)
- [ ] Check cookies for framework hints (PHPSESSID, JSESSIONID, .ASPXAUTH)

---
### Banner Grabbing
```bash
curl -I inlanefreight.com

nc -nv <ip> <port>
```

### WhatWeb 

```shell
whatweb inlanefreight.com 

whatweb -a 3 inlanefreight.com
```
### Wafw00f
```bash
wafw00f inlanefreight.com
```

### [[Nikto]]
```bash
nikto -h inlanefreight.com -Tuning b
```


| Tool         | Description                                                                                                           | Features                                                                                            |
| ------------ | --------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `Wappalyzer` | Browser extension and online service for website technology profiling.                                                | Identifies a wide range of web technologies, including CMSs, frameworks, analytics tools, and more. |
| `BuiltWith`  | Web technology profiler that provides detailed reports on a website's technology stack.                               | Offers both free and paid plans with varying levels of detail.                                      |
| `WhatWeb`    | Command-line tool for website fingerprinting.                                                                         | Uses a vast database of signatures to identify various web technologies.                            |
| `Nmap`       | Versatile network scanner that can be used for various reconnaissance tasks, including service and OS fingerprinting. | Can be used with scripts (NSE) to perform more specialised fingerprinting.                          |
| `wafw00f`    | Command-line tool specifically designed for identifying Web Application Firewalls (WAFs).                             | Helps determine if a WAF is present and, if so, its type and configuration.                         |

---