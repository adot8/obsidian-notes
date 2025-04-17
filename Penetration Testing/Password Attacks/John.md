#### Single Crack Mode
```bash
john --format=<hash_type> <hash or hash_file>

john --format=sha256 hash.txt
john --format=sha256 --wordlist=~/rockyou.txt hash.txt
```
#### Cracked passwords pot file
```bash
cat ~/john/john.pot
```
#### Rules
```bash
john --format=sha256 --wordlist=~/rockyou.txt --rules hash.txt
```
#### Incremental Mode in John
```shell
john --incremental <hash_file>
```
The default character set is limited to `a-zA-Z0-9`
#### Password Protected files
```bash
pdf2john server_doc.pdf > server_doc.hash
john server_doc.hash
```

| Tool                    | Description                                   |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
```bash
locate *2john*
```