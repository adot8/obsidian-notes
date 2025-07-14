
Domain, users, groups, OUs, and GPOs.
```
ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor

ldapsearch (|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456)) --attributes *,ntsecuritydescriptor
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