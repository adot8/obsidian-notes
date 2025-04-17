```bash
*
()
\|
&
```

```bash
PORT    STATE SERVICE VERSION
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: Login
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 149.73 seconds
```

As `OpenLDAP` runs on the server, it is safe to assume that the web application running on port `80` uses LDAP for authentication.

Attempting to log in using a wildcard character (`*`) in the username and password fields grants access to the system, effectively `bypassing any authentication measures that had been implemented`
--------
`LDAP injection` is an attack that `exploits web applications that use LDAP` (Lightweight Directory Access Protocol) for authentication or storing user information. The attacker can `inject malicious code` or `characters` into LDAP queries to alter the application's behaviour, `bypass security measures`, and `access sensitive data` stored in the LDAP directory.

To test for LDAP injection, you can use input values that contain `special characters or operators` that can change the query's meaning:

| Input    | Description                                                                                                                                                                                                                                          |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `*`      | An asterisk `*` can `match any number of characters`.                                                                                                                                                                                                |
| `( )`    | Parentheses `( )` can `group expressions`.                                                                                                                                                                                                           |
| `\|`     | A vertical bar `\|` can perform `logical OR`.                                                                                                                                                                                                        |
| `&`      | An ampersand `&` can perform `logical AND`.                                                                                                                                                                                                          |
| `(cn=*)` | Input values that try to bypass authentication or authorisation checks by injecting conditions that `always evaluate to true` can be used. For example, `(cn=*)` or `(objectClass=*)` can be used as input values for a username or password fields. |

LDAP injection attacks are `similar to SQL injection attacks` but target the LDAP directory service instead of a database.

For example, suppose an application uses the following LDAP query to authenticate users:

Code: php

```php
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```

In this query, `$username` and `$password` contain the user's login credentials. An attacker could inject the `*` character into the `$username` or `$password` field to modify the LDAP query and bypass authentication.

If an attacker injects the `*` character into the `$username` field, the LDAP query will match any user account with any password. This would allow the attacker to gain access to the application with any password, as shown below:

> [!NOTE] Note
> Try `*` for both username and password

Code: php

```php
$username = "*";
$password = "dummy";
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```

Alternatively, if an attacker injects the `*` character into the `$password` field, the LDAP query would match any user account with any password that contains the injected string. This would allow the attacker to gain access to the application with any username, as shown below:

Code: php

```php
$username = "dummy";
$password = "*";
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```
