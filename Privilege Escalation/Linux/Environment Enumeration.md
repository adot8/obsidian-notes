OS information
```bash
cat /etc/os-release
uname -a
lscpu
```

Environment variables
```bash
echo $PATH
env
cat /etc/shells
```

Attached drives, disks, shares, printers(`lpstat`)
```bash
lsblk
cat /etc/fstab
lpstat
```

Mounted and unmounted file systems
```bash
df -h
cat /etc/fstab | grep -v "#" | column -t
```

Network enumeration
```bash
route
netstat -rn
arp -a
cat /etc/resolv.conf
```

User enumeration
```bash
cat /etc/passwd | cut -f1 -d:
grep "*sh$" /etc/passwd
```

Group enumeration - view members of interesting groups with `getent`
```bash
cat /etc/group
getent group sudo
```

Home folders
```bash
ls /home
```

All hidden files + folders for in home folder
```bash
cat /etc/fstab | grep -v "#" | column -t
find / -type d -name ".*" -ls 2>/dev/null
```

View temp folders
```bash
ls -l /tmp /var/tmp /dev/shm
```