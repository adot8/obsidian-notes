
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
```
