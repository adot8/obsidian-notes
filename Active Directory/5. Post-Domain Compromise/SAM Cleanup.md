```bash
cat ntds.hashes | grep ::: | awk -F: '{print $1":"$4}'

cat ntds.hashes | grep '\\' | awk -F\\ '{print $1}' > domain.txt
cat ntds.hashes | grep '\\' | awk -F\\ '{print $2}' > user_hash.txt

cat user_hash.txt | grep '\\' | awk -F\\ '{print $1}' > users.txt
cat user_hash.txt | grep '\\' | awk -F\\ '{print $2}' > hashes.txt
```

```bash
hashcat --user -m 1000 hashes.txt ~/rockyou.txt -O -r /usr/share/hashcat/rules/InsidePro-PasswordsPro.rule

hashcat --user -m 1000 hashes.txt ~/rockyou.txt -O -r /usr/share/hashcat/rules/rockyou-30000.rule

```