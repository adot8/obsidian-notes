The use of file encryption is often still lacking in `private` and `business` matters. Even today, emails containing job applications, account statements, or contracts are often sent unencrypted. This is grossly negligent and, in many cases, even punishable by law.

Especially in business cases, this is quite different for emails. Nowadays, it is pretty common to communicate `confidential` topics or send `sensitive` data `by email`

In many cases, `symmetric encryption` like `AES-256` is used to securely store individual files or folders. Here, the `same key` is used to encrypt and decrypt a file. Therefore, for sending files, `asymmetric encryption` is used, in which `two separate keys` are required. The sender encrypts the file with the `public key` of the recipient. The recipient, in turn, can then decrypt the file using a `private key`.
### Hunting for Encoded Files
```shell
for ext in $(echo ".xls .xls* .xltx .csv .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

```bash
grep -rnw "PRIVATE KEY" /* 2>/dev/null | grep ":1"
```

### Cracking w/ John
```bash
locate *2john*
```

```bash
ssh2john.py SSH.private > ssh.hash
john --wordlist=~/rockyou.txt ssh.hash
john ssh.hash --show
```

```bash
Office2john.py Protected.docx > protected-docx.hash
john --wordlist=~/rockyou.txt protected-docx.hash
john protected-docx.hash --show
```

```bash
pdf2john.py PDF.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
john pdf.hash --show
```