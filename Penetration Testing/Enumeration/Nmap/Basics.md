### Host Discovery
Discover Hosots
```bash
nmap 192.168.1.0/24 -sn  | grep for | cut -d" " -f5
```

> [!NOTE] Note
> `-sn` disables port scanning 

Scan from list
```bash
nmap -sn -iL hosts.list | grep for | cut -d" " -f5
```
### Scan Type

| Flag                     | Type                                                                                       |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| `-sT`                    | TCP Scan (defualt)                                                                         |
| `-sS`                    | TCP SYN Scan                                                                               |
| `-Pn`                    | Disable the ICMP echo requests                                                             |
| `-n`                     | Disables DNS resolution.                                                                   |
| `--disable-arp-ping`     | Disable ARP Scan                                                                           |
| `-sU`                    | UDP Scan                                                                                   |
| `-F`                     | Top 100 ports                                                                              |
| `--reason`               | Determine reason of reponse                                                                |
| `-sV`                    | Service Scan                                                                               |
| `-sA`                    | TCP ACK Scan                                                                               |
| `--packet-trace`         | Shows all packets sent and received.                                                       |
| `-D RND:5`               | Generates five random IP addresses that indicates the source IP the connection comes from. |
| `-e tun0`                | Sends all requests through the specified interface.                                        |
| `--dns-server <ns>,<ns>` | Specify DNS server for Firewall IDS/IPS evaison                                            |
### Saving Output
```bash
nmap 192.168.1.10 -p- -oA nmap.out

xsltproc nmap.xml -o nmap.html
```