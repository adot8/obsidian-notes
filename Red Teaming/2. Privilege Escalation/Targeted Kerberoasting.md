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