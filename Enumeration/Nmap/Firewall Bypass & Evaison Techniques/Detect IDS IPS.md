
> [!NOTE] IMPORTANT
> Using `--packet-trace` is SEVERELY FUNDAMENTAL to understanding Firewall, IDS/IPS rules

Several virtual private servers (`VPS`) with different IP addresses are recommended to determine whether such systems are on the target network during a penetration test

One method to determine whether such `IPS system` is present in the target network is to scan from a single host (`VPS`). If at any time this host is blocked and has no access to the target network, we know that the administrator has taken some security measures. Accordingly, we can continue our penetration test with another `VPS`

### Decoys
The Decoy scanning method (`-D`) generates various random IP addresses inserted into the IP header to disguise the origin of the packet sent.

We can generate random (`RND`) a specific number (for example: `5`) of IP addresses separated by a colon (`:`). Our real IP address is then randomly placed between the generated IP addresses

> [!NOTE] Note
> Another critical point is that the decoys must be alive. Otherwise, the service on the target may be unreachable due to SYN-flooding security mechanisms

```bash
nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

`-D RND:5` = Generates five random IP addresses that indicates the source IP the connection comes from.

We can specify a specific IP to spoof using `-S`. Decoys can be used for SYN, ACK, ICMP scans, and OS detection scans

```bash
nmap 10.129.2.28 -n -Pn -p 445 -O --disable-arp-ping -S 10.129.2.200 -e tun0
```

> [!NOTE] IMPORTANT
> Use this when you get `filtered` for some ports in the output

### DNS Proxying

We can specify DNS servers ourselves (`--dns-server <ns>,<ns>`). This method could be fundamental to us if we are in a demilitarized zone (`DMZ`). The company's DNS servers are usually more trusted than those from the Internet. So, for example, we could use them to interact with the hosts of the internal network.

As another example, we can use `TCP port 53` as a source port (`--source-port`) for our scans. If the administrator uses the firewall to control this port and does not filter IDS/IPS properly, our TCP packets will be trusted and passed through.

```bash
nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --source-port 53
```

Test again using netcat

```bash
nc -nv --source-port 53 10.129.2.28 50000
```