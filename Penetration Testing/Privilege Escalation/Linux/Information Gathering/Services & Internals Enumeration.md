NICs and DNS entires
```bash
ip a
cat /etc/hosts
```

Last logged on users and logged in users 
```bash
lastlog
w
```

History Files
```bash
history
find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null
```

Cron jobs & proc information
```bash
ls -la /etc/cron.daily/
find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"
```

Services
```bash
apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list
```

Binaries
```bash
sudo -V
ls -l /bin /usr/bin/ /usr/sbin/
for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done
```

Configuration files
```bash
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

Scripts
```bash
find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null
```

Running Services by User
```bash
ps aux | grep root
```

Trace System Calls
```bash
strace ping -c1 10.129.112.20
strace suspicious_file
```

