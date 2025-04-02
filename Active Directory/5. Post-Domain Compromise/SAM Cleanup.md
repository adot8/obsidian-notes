```bash
cat ntds.hashes | grep ::: | awk -F: '{print $1":"$4}'
```

```bash
hashcat --user -m 1000 hashes.txt ~/rockyou.txt -O -r /usr/share/hashcat/rules/InsidePro-PasswordsPro.rule
```