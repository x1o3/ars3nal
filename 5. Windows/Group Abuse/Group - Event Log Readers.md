
If [auditing of process creation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation) events and corresponding command line values is enabled. In that case, this information is saved to the Windows security event log as event ID [4688: A new process has been created](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688). Administrators or members of the [Event Log Readers](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255\(v=ws.11\)?redirectedfrom=MSDN#event-log-readers) group have permission to access this log.

---
## Searching Logs for passed Credentials in cmd

We can query Windows events from the command line using the [wevtutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) utility and the [Get-WinEvent](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1) PowerShell cmdlet.

```PowerShell
PS C:\htb> wevtutil qe Security /rd:true /f:text | Select-String "/user"
### Searching Security Logs using wevtutil

C:\htb> wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
### Passing alternate credentials to wevtutil 

PS C:\htb> Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
### We filter for Process creating events (4688) with /user in the process cmd
```

The cmdlet can also be run as another user with the `-Credential` parameter. [PowerShell Operational](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.1) may also contain sensitive information if script block or module logging is enabled. This log is accessible to `unprivileged` users.

```Note
Searching the `Security` event log with `Get-WInEvent` requires administrator access or permissions adjusted on the registry key `HKLM\System\CurrentControlSet\Services\Eventlog\Security`. Membership in just the `Event Log Readers` group is not sufficient.
```

---