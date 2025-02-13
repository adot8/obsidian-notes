Wordpress
```bash
find / -name wp-config.php 2>/dev/null
cat wp-config.php | grep 'DB_USER\|DB_PASSWORD'
```

Find other `config` files
```bash
find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

SSH Keys
```bash
find / -name id_rsa 2>/dev/null
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```