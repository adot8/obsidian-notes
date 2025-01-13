```bash
echo $ip domain | sudo tee -a /etc/hosts
```

```bash
nmap $ip_range -sn  | grep for | cut -d" " -f5

nmap -p- --min-rate=1000 -Pn -v $ip

nmap -p $open_ports -sA -v -n -Pn -disable-arp-ping

nmap -sC -sV -Pn -p $open_ports $ip

nmap -sU -F -v $ip

nmap -p $open_ports --script=vuln -Pn $ip
```

Evaison
```bash
nmap $ip -p $open_ports -sC -sV -v -n -Pn -disable-arp-ping -D RND:5

nmap $ip -p $open_ports -sC -sV -v -n -Pn -disable-arp-ping -S $spoof_host

nmap $ip -p $open_ports -sC -sV -Pn -n --disable-arp-ping --source-port 53
```

Verify scans
```bash
ncat -nv --source-port 53 $ip $port
```