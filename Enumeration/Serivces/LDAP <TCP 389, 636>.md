Dump domain info
```bash
ldapdomaindump ldap://dc.sequel.htb -u 'sequel.htb\user' -p 'pwd'
ldapsearch -H ldaps://dc.sequel.htb -D 'user@sequel.htb' -w 'pwd' -b 'dc=sequel,dc=htb'   
```
Null Authentication
```bash
 ldapsearch -H ldap://hutchdc.hutch.offsec -D '' -w '' -b "dc=hutch,dc=offsec"
  ldapsearch -H ldap://hutchdc.hutch.offsec -D '' -w '' -b "dc=hutch,dc=offsec" | grep description
```
Netexec usage
```bash
netxec ldap <IP> -u '' -p '' --password-not-required --admin-count --users --groups
```
Viewing the certificate with openssl can hint towards the domain controller being a CA.
```bash
openssl s_client -showcerts -connect 10.10.11.202:3269
```
![[Pasted image 20250212120035.png]]

```bash
ldapsearch -H ldap://ldap.example.com:389 -D "cn=admin,dc=example,dc=com" -w secret123 -b "ou=people,dc=example,dc=com" "(mail=john.doe@example.com)"
```
This command can be broken down as follows:

- Connect to the server `ldap.example.com` on port `389`.
- Bind (authenticate) as `cn=admin,dc=example,dc=com` with password `secret123`.
- Search under the base DN `ou=people,dc=example,dc=com`.
- Use the filter `(mail=john.doe@example.com)` to find entries that have this email address.
### LDAP Injection
`LDAP injection` is an attack that `exploits web applications that use LDAP` (Lightweight Directory Access Protocol) for authentication or storing user information. The attacker can `inject malicious code` or `characters` into LDAP queries to alter the application's behaviour, `bypass security measures`, and `access sensitive data` stored in the LDAP directory.

To test for LDAP injection, you can use input values that contain `special characters or operators` that can change the query's meaning:

|Input|Description|
|---|---|
|`*`|An asterisk `*` can `match any number of characters`.|
|`( )`|Parentheses `( )` can `group expressions`.|
|`\|`|A vertical bar `\|` can perform `logical OR`.|
|`&`|An ampersand `&` can perform `logical AND`.|
|`(cn=*)`|Input values that try to bypass authentication or authorisation checks by injecting conditions that `always evaluate to true` can be used. For example, `(cn=*)` or `(objectClass=*)` can be used as input values for a username or password fields.|

LDAP injection attacks are `similar to SQL injection attacks` but target the LDAP directory service instead of a database.

For example, suppose an application uses the following LDAP query to authenticate users:

Code: php

```php
(&(objectClass=user)(sAMAccountName=$username)(userPassword=$password))
```

In this query, `$username` and `$password` contain the user's login credentials. An attacker could inject the `*` character into the `$username` or `$password` field to modify the LDAP query and bypass authentication.

If an attacker injects the `*` character into the `$username` field, the LDAP query will match any user account with any password. This would allow the attacker to gain access to the application with any password, as shown below:

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


### Overview
`LDAP` (Lightweight Directory Access Protocol) is `a protocol` used to `access and manage directory information`. A `directory` is a `hierarchical data store` that contains information about network resources such as `users`, `groups`, `computers`, `printers`, and other devices

Differences between LDAP protocol and it's role in Active Directory

|**LDAP**|**Active Directory (AD)**|
|---|---|
|A `protocol` that defines how clients and servers communicate with each other to access and manipulate data stored in a directory service.|A `directory server` that uses LDAP as one of its protocols to provide authentication, authorisation, and other services for Windows-based networks.|
|An `open and cross-platform protocol` that can be used with different types of directory servers and applications.|`Proprietary software` that only works with Windows-based systems and requires additional components such as DNS (Domain Name System) and Kerberos for its functionality.|
|It has a `flexible and extensible schema` that allows custom attributes and object classes to be defined by administrators or developers.|It has a `predefined schema` that follows and extends the X.500 standard with additional object classes and attributes specific to Windows environments. Modifications should be made with caution and care.|
|Supports `multiple authentication mechanisms` such as simple bind, SASL, etc.|It supports `Kerberos` as its primary authentication mechanism but also supports NTLM (NT LAN Manager) and LDAP over SSL/TLS for backward compatibility.|
