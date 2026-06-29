
We can check for `SQL Admin Rights` in the `Node Info` tab in `BloodHound` for a given user or use this custom `Cypher query` to search:

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

---

### Enumerating Instances with PowerUpSQL

```powershell
PS C:\htb>  Import-Module .\PowerUpSQL.ps1
PS C:\htb>  Get-SQLInstanceDomain
```

### Queries with PowerUpSQL

```powershell
PS C:\htb>  Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

---

### MSSQLCLIENT

```shell
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```

We can run commands in the format `xp_cmdshell <command>`. Here we can enumerate the rights that our user has on the system.

---