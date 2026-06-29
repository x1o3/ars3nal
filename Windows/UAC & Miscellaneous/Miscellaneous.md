
## Table of Contents:

- [[#LOLBANS]]
- [[#Windows Certificate Dialog CVE-2019-1388|CVE-2019-1388]]

---
## LOLBANS

[LOLBAS project](https://lolbas-project.github.io/) documents binaries, scripts, and libraries that can be used for "living off the land" techniques on Windows systems.

---

## Windows Certificate Dialog CVE-2019-1388

[CVE-2019-1388](https://nvd.nist.gov/vuln/detail/CVE-2019-1388) is a privilege escalation vulnerability in the Windows Certificate Dialog, which did not properly enforce user privileges.
### Exploitation

The issue was in the UAC mechanism, with an option to show information about an executable's certificate, opening the Windows certificate dialog when a user clicks the link. The `Issued By` field in the General tab is rendered as a hyperlink if the binary is signed with a certificate that has Object Identifier (OID) `1.3.6.1.4.1.311.2.1.10`. This OID value is identified in the [wintrust.h](https://docs.microsoft.com/en-us/windows/win32/api/wintrust/) header as [SPC_SP_AGENCY_INFO_OBJID](https://docs.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptformatobject) which is the `SpcSpAgencyInfo` field in the details tab of the certificate dialog. If it is present, a hyperlink in the field will render in the General tab. This vulnerability can be exploited easily using an old Microsoft-signed executable ([hhupd.exe](https://packetstormsecurity.com/files/14437/hhupd.exe.html)) that's `Run as administrator` and contains a certificate with the `SpcSpAgencyInfo` field populated with a hyperlink.

When we click on the hyperlink, a browser window will launch running as `NT AUTHORITY\SYSTEM`. Once the browser is opened, it is possible to "break out" of it by leveraging the `View page source` menu option and then `save as` on the machine to launch a PS or cmd console as SYSTEM through the file path field.

This [link](https://web.archive.org/web/20210620053630/https://gist.github.com/gentilkiwi/802c221c0731c06c22bb75650e884e5a) lists all of the vulnerable Windows Server and Workstation versions.

---