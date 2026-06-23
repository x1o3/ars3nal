
## Attack Surface

### Passive Web Recon
- [ ] [[#Google Dorking]]
- [ ] [[#GitHub Dorking]]
- [ ] [[#Wayback Machine]]
- [ ] [[#Certificate Transparency]]
- [ ] [[#Shodan]]
- [ ] [[#DNS History & Passive DNS]]
- [ ] [[#ASN & IP Space Mapping]]

### People & Organisation Intel
- [ ] [[#Company Structure & Acquisitions]]
- [ ] [[#Email Harvesting]]
- [ ] [[#Breach Data]]

---

## Google Dorking

In-Depth: [[Google Dorking]]

- [ ] Find exposed files: `site:target.com filetype:env OR filetype:conf OR filetype:log`
- [ ] Find login/admin panels: `site:target.com inurl:admin OR inurl:login OR inurl:dashboard`
- [ ] Find directory listings: `site:target.com intitle:"index of"`
- [ ] Find exposed credentials: `site:target.com filetype:env "DB_PASSWORD"`
- [ ] Find registration pages: `site:target.com inurl:signup OR inurl:register`
- [ ] Find config files: `site:target.com ext:xml OR ext:conf OR ext:cfg OR ext:ini`
- [ ] Find subdomains: `site:*.target.com -www`
- [ ] Find cached/old pages: `cache:target.com`
- [ ] Find mentions on other sites: `"target.com" -site:target.com`

Use Google's verbatim search mode, auto-correct can silently modify dorks

---

## GitHub Dorking

In-Depth: [[GitHub Dorking]]

- [ ] Search org for secrets: `org:targetorg "api_key" OR "secret" OR "password"`
- [ ] Search for internal URLs: `org:targetorg "internal.target.com"`
- [ ] Search for connection strings: `org:targetorg "DB_CONNECTION" OR "mongodb://" OR "mysql://"`
- [ ] Search for AWS keys: `org:targetorg "AKIA"`
- [ ] Search for private keys: `org:targetorg "BEGIN RSA PRIVATE KEY"`
- [ ] Check commit history of interesting repos as secrets may get deleted but history remains
- [ ] Check gists from company employee accounts
- [ ] Search Pastebin/paste sites for company domain
- [ ] Search employee names/emails directly as personal repos often have company secrets

---

## Wayback Machine

- [ ] Search target on web.archive.org
- [ ] Pull all historical URLs: `waybackurls target.com | tee wayback.txt`
- [ ] Filter for interesting endpoints: `cat wayback.txt | grep "="`
- [ ] Filter for old API endpoints: `cat wayback.txt | grep "/api/"`
- [ ] Filter for backup files: `cat wayback.txt | grep -E "\.bak|\.old|\.zip|\.tar"`
- [ ] Check old JS files for endpoints that still exist
- [ ] Look for pages that returned 200 historically but 404 now — may still be live
- [ ] Use gau (GetAllUrls) alongside waybackurls, it pulls from multiple sources including Common Crawl

---

## Certificate Transparency

CT logs record every certificate ever issued

- [ ] Search crt.sh: `https://crt.sh/?q=%.target.com`
- [ ] Pull via curl: `curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value'`
- [ ] Filter wildcards and clean list: `sed 's/\*\.//g' | sort -u`
- [ ] Cross-reference with active subdomain scan results
- [ ] Check for internal-looking subdomains (dev, staging, internal, vpn)
- [ ] Search by organisation name not just domain as crt.sh lets you search by O= field in cert

---

## Shodan

- [ ] Search by domain: `hostname:target.com`
- [ ] Search by ASN: `asn:XXXXX`
- [ ] Filter by title: `asn:XXXXX http.title:"Dashboard"`
- [ ] Check Facet Analysis for top ports on ASN
- [ ] Look for exposed management interfaces: Grafana, Kibana, Jenkins, PRTG
- [ ] Check for CVEs Shodan has flagged on target IPs
- [ ] Search for default pages: `http.title:"Welcome to nginx"`
- [ ] Filter by product: `org:"Target Inc" product:"Apache httpd"`
- [ ] Search by IP range if you have the ASN, `net:1.2.3.0/24`

---

## DNS History & Passive DNS

- [ ] Check `SecurityTrails` for DNS history
- [ ] Check `ViewDNS.info` for historical A records
- [ ] Look for origin IP before CDN was added
- [ ] Check MX records as mail servers often reveal real IP ranges
- [ ] Check old SPF records for internal IP ranges
- [ ] SPF records frequently contain internal mail relay IPs: `dig TXT target.com`

---
## ASN & IP Space Mapping

Map the full IP space before active scanning

- [ ] Find ASN via `bgp.he.net` or `asnlookup`
- [ ] Run `metabigor` for all IP ranges under ASN
- [ ] Search [[Shodan]] by ASN: asn:XXXXX
- [ ] Filter [[Shodan]] by http.title for exposed panels
- [ ] Check Facet Analysis in [[Shodan]] for top open ports
- [ ] Cross-reference IP ranges with `crt.sh` subdomain results

---

## Company Structure & Acquisitions

- [ ] Search `Crunchbase` for acquisitions and subsidiaries
- [ ] Map all acquired companies, find their domains
- [ ] Check `LinkedIn` for employee count, tech stack mentions, job postings
- [ ] Job postings reveal tech stack: "experience with Jenkins, AWS, Kubernetes"
- [ ] Check Companies House / SEC filings for subsidiaries

---

## Email Harvesting

- [ ] `theHarvester` for passive email collection
- [ ] `Hunter.io` for company email format and addresses
- [ ] Check LinkedIn for employee names and derive emails from format
- [ ] Search GitHub commits for company email addresses
- [ ] Cross-reference with breach data

Look for Email format (first.last vs flast), valid employee names for spraying.

---

## Breach Data

- [ ] Search HaveIBeenPwned for domain
- [ ] Check DeHashed for company email domain
- [ ] Cross-reference found credentials against current login portals
- [ ] Check for password patterns — people reuse base passwords with variations

---
