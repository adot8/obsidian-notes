## Relational Databases
Store structured data in tables with rows and columns, and use SQL for querying. Relational databases are ideal for applications that require structured data with a high level of integrity, such as complex queries
## Non-relational databases
Also known as NoSQL databases, these databases use a variety of data models to store unstructured or semi-structured data. Non-relational databases are ideal for applications that require storing large volumes of data, low latency, and flexible data models.

> [!NOTE] Note
> The data can be stored in key:value pairs just like json

## MySQL
```shell
mysql- u root -p<password>
mysql -u root -h docker.hackthebox.eu -P 3306 -p<password>
```
### Create database
```sql
create DATABASE users;
SHOW DATABASES;
USE users;
```
### Create table
```sql
CREATE TABLE logins (
    id INT,
    username VARCHAR(100),
    password VARCHAR(100),
    date_of_joining DATETIME
    );

SHOW TABLES;
DESCRIBE logins;
```

> [!NOTE] Note
> The first column, `id` is an integer. The following two columns, `username` and `password` are set to strings of 100 characters each. Any input longer than this will result in an error. The `date_of_joining` column of type `DATETIME` stores the date when an entry was added

### Table properties
```SQL
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT,
    username VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(100) NOT NULL,
    date_of_joining DATETIME DEFAULT NOW(),
    PRIMARY KEY (id)
    );
```
- `NOT NULL`
	- column can't be left empty
- `AUTO_INCREMENT`
	- automatically increments the field by 1 every time a new item is added to the table
- `DEFAULT`
	- Specifies default value. For example, within the `date_of_joining` column, we can set the default value to [Now()](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html#function_now), which in MySQL returns the current date and time
- `PRIMARY KEY`
### Insert Statement
```sql
INSERT INTO table_name VALUES (column1_value, column2_value, column3_value, ...);
INSERT INTO logins VALUES(1, 'admin', 'p@ssw0rd', '2020-07-02');

INSERT INTO table_name(column2, column3) VALUES (column2_value, column3_value);
INSERT INTO logins(username, password) VALUES('administrator', 'adm1n_p@ss');

INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');

```
### Select Statement
```SQL
SELECT * FROM table_name;
SELECT * FROM logins;

SELECT column1, column2 FROM table_name;
SELECT username,password FROM logins;
```
### Drop Statement
Remove tables from the database. This is **PERMANENT**
```SQL
DROP TABLE table_name;
DROP TABLE logins;
```
### Alter Statement
Change name of table or any of it's fields, delete/add new columns 
```sql
ALTER TABLE table_name ADD newColumn_name value_type(int,varchar,etc.);
ALTER TABLE logins ADD age INT;

ALTER TABLE table_name RENAME COLUMN oldColumn TO newColumn;
ALTER TABLE logins RENAME COLUMN age TO age2;

ALTER TABLE table_name MODIFY column_name data_type;
ALTER TABLE logins MODIFY age VARCHAR(100);

ALTER TABLE table_name DROP column_name;
ALTER TABLE logins DROP age;
```
### Update Statement
Update records/values within table columns
```sql
UPDATE table_name SET column1=newvalue1, column2=newvalue2, ... WHERE <condition>

mysql> UPDATE logins SET password = 'change_password' WHERE id > 1;
mysql> SELECT * FROM logins;

+----+---------------+-----------------+---------------------+
| id | username      | password        | date_of_joining     |
+----+---------------+-----------------+---------------------+
|  1 | admin         | p@ssw0rd        | 2020-07-02 00:00:00 |
|  2 | administrator | change_password | 2020-07-02 11:30:50 |
|  3 | john          | change_password | 2020-07-02 11:47:16 |
|  4 | tom           | change_password | 2020-07-02 11:47:16 |
+----+---------------+-----------------+---------------------+
```
### Sorting Results
#### Order By
Sort results of any query using `ORDER BY`. By default it sorts by ascending order (ASC) this can be changed to descending (DESC).
```SQL
SELECT * FROM table_name ORDER BY column_name;
SELECT * FROM table_name ORDER BY column_name DESC;
SELECT * FROM logins ORDER BY password;
SELECT * FROM logins ORDER BY password DESC;
```
#### LIMIT results
In case our query returns a large number of records, we can [LIMIT](https://dev.mysql.com/doc/refman/8.0/en/limit-optimization.html) the results to what we want only, using `LIMIT` and the number of records we want
```sql
 mysql> SELECT * FROM logins LIMIT 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
+----+---------------+------------+---------------------+
```
If we wanted to LIMIT results with an offset, we could specify the offset before the LIMIT count
```sql
mysql> SELECT * FROM logins LIMIT 1, 2;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
```
#### WHERE Clause
To filter or search for specific data, we can use conditions with the SELECT statement using the WHERE clause, to fine-tune the results
```sql
SELECT * FROM table_name WHERE <condition>;

mysql> SELECT * FROM logins WHERE id > 1;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
```
```sql
mysql> SELECT * FROM logins where username = 'admin';

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  1 | admin    | p@ssw0rd | 2020-07-02 00:00:00 |
+----+----------+----------+---------------------+
```
#### LIKE Clause
Another useful SQL clause is [LIKE](https://dev.mysql.com/doc/refman/8.0/en/pattern-matching.html), enabling selecting records by matching a certain pattern. The query below retrieves all records with usernames starting with `admin`
```sql
mysql> SELECT * FROM logins WHERE username LIKE 'admin%';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  4 | administrator | adm1n_p@ss | 2020-07-02 15:19:02 |
+----+---------------+------------+---------------------+
```

> [!NOTE] NOTE
> The % symbol acts as a wildcard and matches all characters after admin

Match records based on number of characters using an`_`
```sql
mysql> SELECT * FROM logins WHERE username like '___';

+----+----------+----------+---------------------+
| id | username | password | date_of_joining     |
+----+----------+----------+---------------------+
|  3 | tom      | tom123!  | 2020-07-02 15:18:56 |
+----+----------+----------+---------------------+
```
### SQL Operators
#### AND Operator
The `AND` operator takes in two conditions and returns `true` or `false` based on their evaluation
```sql
condition1 AND condition2
```
The result of the `AND` operation is `true` if and only if both `condition1` and `condition2` evaluate to `true`
```sql
mysql> SELECT 1 = 1 AND 'test' = 'test';

+---------------------------+
| 1 = 1 AND 'test' = 'test' |
+---------------------------+
|                         1 |
+---------------------------+

mysql> SELECT 1 = 1 AND 'test' = 'abc';

+--------------------------+
| 1 = 1 AND 'test' = 'abc' |
+--------------------------+
|                        0 |
+--------------------------+
```

> [!NOTE] Note
> Any `non-zero` value is considered `true`, and it usually returns the value `1` to signify `true`. `0` is considered `false`
#### OR Operator
The `OR` operator takes in two expressions as well, and returns `true` when at least one of them evaluates to `true`
```sql
mysql> SELECT 1 = 1 OR 'test' = 'abc';

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
The `NOT` operator simply toggles a `boolean` value 'i.e. `true` is converted to `false` and vice versa. 
```sql
```shell-session
mysql> SELECT NOT 1 = 1;

+-----------+
| NOT 1 = 1 |
+-----------+
|         0 |
+-----------+

mysql> SELECT NOT 1 = 2;

+-----------+
| NOT 1 = 2 |
+-----------+
|         1 |
+-----------+
```
the first query resulted in `false` because it is the inverse of the evaluation of `1 = 1`, which is `true`, so its inverse is `false`
#### Symbol Operators
`AND`, `OR` and `NOT` operators can also be represented as `&&`, `||` and `!`
#### Using Operators in queries
```sql
mysql> SELECT * FROM logins WHERE username != 'john';

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+

mysql> SELECT * FROM logins WHERE username != 'john' AND id > 1;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  4 | tom           | tom123!    | 2020-07-02 11:47:16 |
+----+---------------+------------+---------------------+
```

> [!NOTE] NOTE
> Math is also possible in MySQL with the following order of precedence
> 
- Division (`/`), Multiplication (`*`), and Modulus (`%`)
- Addition (`+`) and subtraction (`-`)
-----------------------------------------
- Comparison (`=`, `>`, `<`, `<=`, `>=`, `!=`, `LIKE`)
------------------------------------------------------------------------
- NOT (`!`)
- AND (`&&`)
- OR (`||`)
```sql
select * from titles where title NOT LIKE '%Engineer';

select * from titles where emp_no > 10000 OR title NOT LIKE '%engineer';
```