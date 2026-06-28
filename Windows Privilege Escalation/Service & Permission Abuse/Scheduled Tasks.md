
We can use the [schtasks](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks) command or [Get-ScheduledTask](https://docs.microsoft.com/en-us/powershell/module/scheduledtasks/get-scheduledtask?view=windowsserver2019-ps) PowerShell cmdlet to enumerate scheduled tasks on the system. By default, we can only see tasks created by our user and default scheduled tasks that every Windows operating system has.

```PowerShell
C:\htb> schtasks /query /fo LIST /v

PS C:\htb> Get-ScheduledTask | select TaskName,State
### Enumerating with PowerShell
```

We can check permissions and try to exploit them. Scheduled tasks created by other users (such as admins) are stored in `C:\Windows\System32\Tasks`. 

---