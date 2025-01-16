### Input Sanitization
Escape special characters with `mysqli_real_escape_string()` function

> [!NOTE] Note
> `pg_escape_string()` can be used for PostgresSQL

Before:
```php
$username = $_POST['username'];
$password = $_POST['password'];
```
After:
```php
$username = mysqli_real_escape_string($conn, $_POST['username']);
$password = mysqli_real_escape_string($conn, $_POST['password']);
```
### Input Validation
Restrict users input to only fit a certain format (emails for example).
Before:
```php
<?php
if (isset($_GET["port_code"])) {
	$q = "Select * from ports where port_code ilike '%" . $_GET["port_code"] . "%'";
	$result = pg_query($conn,$q);
	if (!$result)
	{
   		die("</table></div><p style='font-size: 15px;'>" . pg_last_error($conn). "</p>");
	}
```
After (letters and spaces only):
```php
$pattern = "/^[A-Za-z\s]+$/";
$code = $_GET["port_code"];

if(!preg_match($pattern, $code)) {
  die("</table></div><p style='font-size: 15px;'>Invalid input! Please try again.</p>");
}

$q = "Select * from ports where port_code ilike '%" . $code . "%'";
```

> [!NOTE] Note
> `preg_match` checks if the input matches the given pattern or not

### Others
Make sure user privileges are following least privilege and set the `secure_file_priv` to NULL.
Get a WAF lol
