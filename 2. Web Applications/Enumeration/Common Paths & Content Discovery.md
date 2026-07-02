
## Common Paths

- [ ] Auth endpoints: /login /log_in /sign_up /signup /logout /log_out
- [ ] User endpoints: /users /users/new /users/1
- [ ] Admin endpoints: /admin /admin_panel
- [ ] API endpoints: /api /api/v1 /api/v2 /API /soap
- [ ] Try site.example.com/site and site.example.com/example (subdomain name as path)
- [ ] Check all above for both GET and POST methods
- [ ] /`robots.txt` /`sitemap.xml` /`wpscan`
- [ ] Try [[Broken Access Control#HTTP verb tampering|HTTP verb tampering]] on endpoints returning 405 as PUT/PATCH/ DELETE may be open.


---

## Content Discovery

### Sensitive Information

Look source code for sensitive information (comments, exposed code etc.)

### FinalRecon

A web crawling and recon tool that automates information gathering for a target domain, including subdomains, DNS records, WHOIS data, SSL certificates, headers, and technology detection.
```bash
./finalrecon.py --url <website> --full
```

### ReconSpider

A web crawling and recon tool that spiders a website to collect URLs, endpoints, JavaScript files, forms, emails, comments, and other potentially interesting information.
```bash
python3 ReconSpider.py http://inlanefreight.com
### Data is saved in a JSON file.
```

---

## Error Page Fingerprinting

- [ ] Check what `404` looks like: /.404 /404.html /404.php /%00%01
- [ ] Check what `403` looks like for if is it a WAF or a real app deny?
- [ ] Check what `500` looks like for triggers with malformed input
- [ ] Note if error pages are custom or default (default = more info leakage)
- [ ] Check if `404` vs `200` is distinguishable (for fuzzing accuracy)
- [ ]  Try /%00%01 and other null/invalid byte paths

---