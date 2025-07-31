
> Default post-ex commands will be ran by `svchost.exe` after malleable C2 configs

For any outbound connection commands find an msedge or chrome ppid change the spawnto to msedge.exe or chrome.exe


#### HTTP

Find msedge process and inject shell code into it via GUI

```powershell
spawnto x64 "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
powerpick command
```


#### SMB W32TIME_ALT (LDAP)

```powershell
spawnto x64 C:\Windows\System32\svchost.exe
```

```powershell
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe AntiVirus
```

