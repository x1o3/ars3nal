
## Identifying Users

```shell
> kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
  ### Kerbrute - Internal AD Username Enumeration

> rpcclient -U'%' 10.10.110.17
  rpcclient $> enumdomusers
  ### with rpcclient
  
> enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
  ### with enum4linux, also have the enum4linux-ng version
  
> nxc smb 172.16.5.5 --users 
  ### with netexec 
  
> ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
  ### with ldapsearch 
  
> ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
  ### We can use this with valid credentials to perform the same enumeration 
```

---
## Hosts
### Wireshark + ARP Filter to Discover Hosts

We can also look at `MDNS` protocol to see full `FQDN` of hosts.

```shell
tcpdump -i ens224

fping -asgq 172.16.5.0/23
```

---