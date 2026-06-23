
## Attack Surface

- [ ] [[#JS File Discovery]]
- [ ] [[#Endpoint & API Extraction]]
- [ ] [[#Secret & Credential Hunting]]
- [ ] [[#Source Map Exploitation]]
- [ ] [[#Webpack & Bundle Analysis]]
- [ ] [[#PostMessage Analysis]]
- [ ] [[#Prototype Pollution Hunting]]

---

## JS File Discovery

- [ ] Spider target with Burp and filter for `.js` files in sitemap
- [ ] Use subjs to pull JS files from a list of domains
- [ ] Use GetJS to extract JS files from a single target
- [ ] Check for JS files in page source manually
- [ ] Pull historical JS files from Wayback Machine for old versions leak more
- [ ] Check for JS files on subdomains not just main domain
- [ ] Look for lazy-loaded JS files that don't appear on first load
- [ ] Check for JS files loaded dynamically and interact with the app features and watch Burp for new JS requests

```bash
# subjs
echo "https://target.com" | subjs

# getJS
getJS --url https://target.com --complete

# katana for JS discovery
katana -u https://target.com -jc -d 3

# waybackurls for historical JS
waybackurls target.com | grep "\.js$" | sort -u

# pulling JS from multiple subdomains
cat subdomains.txt | subjs | tee js-files.txt
```

---

## Endpoint & API Extraction

- [ ] Run `LinkFinder` against all discovered JS files
- [ ] Run `xnLinkFinder` for deeper endpoint extraction
- [ ] Grep for API patterns manually
- [ ] Look for versioned API endpoints `/api/v1` `/api/v2`
- [ ] Check for internal hostnames and URLs hardcoded in JS
- [ ] Look for `GraphQL` endpoints
- [ ] Feed discovered endpoints back into ffuf for fuzzing

```bash
# LinkFinder on single file
python3 linkfinder.py -i https://target.com/app.js -o cli

# LinkFinder on entire domain
python3 linkfinder.py -i https://target.com -d -o results.html

# xnLinkFinder
python3 xnLinkFinder.py -i https://target.com -d 10 -o endpoints.txt

# Manual grep patterns
cat app.js | grep -E "(\/api\/|\/v[0-9]\/|\/graphql|\/rest\/)"
cat app.js | grep -E "https?://[a-zA-Z0-9./?=_-]*"
cat app.js | grep -E "(fetch|axios|XMLHttpRequest|ajax)\s*\("

# Extract all paths
cat js-files.txt | while read url; do
  curl -s "$url" | grep -oE '"\/[a-zA-Z0-9/_-]+"' 
done
```

Tip: Beautify/deobfuscate the JS first as minified code kills grep accuracy.

---

## Secret & Credential Hunting

- [ ] Run `JShole` against target
- [ ] Run` SecretFinder` on all JS files
- [ ] Grep for common secret patterns manually
- [ ] Check for hardcoded API keys (AWS, Google, Stripe, Twilio)
- [ ] Look for `JWT secrets` and signing keys
- [ ] Check for hardcoded credentials (username/password combos)
- [ ] Look for internal IP addresses and hostnames
- [ ] Check for S3 bucket names and GCS bucket references
- [ ] Search for debug/test credentials left in production

```bash
# SecretFinder
python3 SecretFinder.py -i https://target.com/app.js -o cli

# SecretFinder on entire domain
python3 SecretFinder.py -i https://target.com -e -o results.html

# JShole
python3 jshole.py -u https://target.com

# Manual grep for common secrets
cat app.js | grep -E "(api_key|apikey|api-key|secret|password|passwd|token|auth)" -i
cat app.js | grep -E "AKIA[0-9A-Z]{16}"  # AWS keys
cat app.js | grep -E "AIza[0-9A-Za-z-_]{35}"  # Google API
cat app.js | grep -E "sk_live_[0-9a-zA-Z]{24}"  # Stripe
cat app.js | grep -E "xox[baprs]-[0-9]{12}"  # Slack
cat app.js | grep -E "ghp_[a-zA-Z0-9]{36}"  # GitHub tokens
cat app.js | grep -E "[0-9a-f]{32}"  # Generic 32char hex secrets
cat app.js | grep -iE "(jdbc|mongodb|mysql|postgres|redis):\/\/"  # DB strings

# Internal IPs
cat app.js | grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}"
```


---

## Source Map Exploitation

- [ ] Check if `.map` files are publicly accessible
- [ ] Look for `//# sourceMappingURL=` comment at bottom of JS files
- [ ] Download and extract source maps
- [ ] Use `source-map-explorer` or `unwebpack` to reconstruct source
- [ ] Read reconstructed source for secrets, logic, endpoints

```bash
# Check for source map reference
curl -s https://target.com/app.js | tail -5
# Look for: //# sourceMappingURL=app.js.map

# Try fetching the map file directly
curl -s https://target.com/app.js.map -o app.js.map

# Check common map file locations
curl -I https://target.com/static/js/main.chunk.js.map
curl -I https://target.com/js/bundle.js.map

# Extract source from map file
npm install -g source-map-explorer
source-map-explorer app.js app.js.map

# Reconstruct files from map
node -e "
const fs = require('fs');
const map = JSON.parse(fs.readFileSync('app.js.map'));
map.sources.forEach((src, i) => {
  console.log(src);
});
"
```


---

## Webpack & Bundle Analysis

- [ ] Identify webpack chunks and look for `chunk.js` files in source
- [ ] Find the webpack runtime file, they may contains module registry
- [ ] Use `relative-url-extractor` on bundle files
- [ ] Look for lazy-loaded route definition, they reveals all app routes
- [ ] Check for feature flags and hidden functionality
- [ ] Look for commented-out debug endpoints

```bash
# Find all webpack chunks
curl -s https://target.com | grep -oE "\"[^\"]+\.chunk\.js\"" 

# Extract URLs from bundle
python3 relative-url-extractor.py app.bundle.js

# Download and analyze all chunks
cat chunks.txt | while read chunk; do
  curl -s "https://target.com/$chunk" >> full-bundle.js
done

# Find route definitions in React apps
cat full-bundle.js | grep -E "(path|route):\s*[\"']\/[a-zA-Z]"

# Find lazy loaded components
cat full-bundle.js | grep -E "import\s*\("
```

---

## PostMessage Analysis

- [ ] Search JS for `postMessage` calls
- [ ] Search for message event listeners
- [ ] Identify origin validation if it is strict or missing?
- [ ] Check what data is passed in messages
- [ ] Test for `XSS` via `postMessage` with missing origin check
- [ ] Check for sensitive data leaked via `postMessage`
- [ ] Open browser `DevTools > Sources > Global Listeners > message`, it shows all registered message handlers

```bash
# Find postMessage usage
cat app.js | grep -E "postMessage\s*\("
cat app.js | grep -E "addEventListener\s*\([\"']message[\"']"

# Check origin validation
cat app.js | grep -A5 "addEventListener.*message" | grep -E "(origin|source)"

# Look for data being passed
cat app.js | grep -B2 -A10 "postMessage"
```


---

## Prototype Pollution Hunting

- [ ] Check JS for vulnerable merge/clone patterns
- [ ] Test `__proto__` and `constructor.prototype` in parameters
- [ ] Use `PPScan` to automate detection
- [ ] Check query string parsers for pollution
- [ ] Test JSON body parameters
- [ ] Look for gadget chains that lead to XSS or RCE
- [ ] Check the blackFan client-side prototype pollution repo, it has gadgets for most common frameworks

```bash
# PPScan
node ppScan.js -u https://target.com

# Manual test payloads in URL params
https://target.com/?__proto__[test]=polluted
https://target.com/?constructor[prototype][test]=polluted

# Check with browser console after request
Object.prototype.test  # should return undefined if not polluted

# Grep for vulnerable patterns in JS
cat app.js | grep -E "(merge|extend|clone|assign)\s*\("
cat app.js | grep -E "(\[\s*['\"]__proto__['\"]|\[constructor\])"

# DOM-based gadget check
cat app.js | grep -E "(innerHTML|document\.write|eval)\s*\("
```

---