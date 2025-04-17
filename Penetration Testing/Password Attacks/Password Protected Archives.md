Besides standalone files, there is also another format of files that can contain not only data, such as an Office document or a PDF, but also other files within them. This format is called an `archive` or `compressed file` that can be protected with a password if necessary

Compressed file type wordlist
```bash
curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt
```

### Cracking Archives
```bash
zip2john ZIP.zip > zip.hash
john --wordlist=rockyou.txt zip.hash
john zip.hash --show
```

### Cracking OpenSSL Encrypted Archives
When cracking OpenSSL encrypted files and archives, we can encounter many different difficulties that will bring many false positives or even fail to guess the correct password. Therefore, the safest choice for success is to use the `openssl` tool in a `for-loop` that tries to extract the files from the archive directly if the password is guessed correctly

```shell
adot8@htb[/htb]$ file GZIP.gzip 

GZIP.gzip: openssl enc'd data with salted password
```

```shell
for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

### Cracking BitLocker Encrypted Drives
```shell
bitlocker2john -i Backup.vhd > backup.hashes
grep "bitlocker\$0" backup.hashes > backup.hash
hashcat -m 22100 backup.hash ~/rockyou.txt -o backup.cracked
```

Mount
```bash
mkdir mount/
modprobe nbd
qemu-nbd -c /dev/nbd0 <PATH_TO_FILE>.vhd
cryptsetup bitlkOpen /dev/nbd0p2 bitlocker_backup
mount /dev/mapper/bitlockerbackup mount/
```

Close
```bash
umount mount
cryptsetup bitlkClose bitlocker_backup
```
https://medium.com/@kartik.sharma522/mounting-bit-locker-encrypted-vhd-files-in-linux-4b3f543251f0