```powershell
nmap $ip -p $open_ports -sC -sV -v -n -Pn -disable-arp-ping -D RND:5

nmap $ip -p $open_ports -sC -sV -v -n -Pn -disable-arp-ping -S $spoof_host

nmap $ip -p $open_ports -sC -sV -Pn -n --disable-arp-ping --source-port 53
```