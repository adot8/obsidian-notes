```bash
cat ntds.hashes | grep ::: | awk -F: '{print $1":"$4}'

cat ntds.hashes | grep '\\' | awk -F\\ '{print $1}'
cat ntds.hashes | grep '\\' | awk -F\\ '{print $2}' > user_hash.txt
```

```bash
hashcat --user -m 1000 hashes.txt ~/rockyou.txt -O -r /usr/share/hashcat/rules/InsidePro-PasswordsPro.rule

hashcat --user -m 1000 hashes.txt ~/rockyou.txt -O -r /usr/share/hashcat/rules/rockyou-30000.rule

```