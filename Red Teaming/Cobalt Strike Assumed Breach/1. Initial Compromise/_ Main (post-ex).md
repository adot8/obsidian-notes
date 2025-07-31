
> Default post-ex commands will be ran by `svchost.exe` after malleable C2 configs

For any outbound connection commands find an msedge or chrome ppid change the spawnto to msedge.exe or chrome.exe

```powershell
ppid 6648
spawnto x64 "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
powerpick command
```