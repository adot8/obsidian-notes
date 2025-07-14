
>PsExec is generally regarded as one of the loudest lateral movement techniques as new service creations are relatively rare events to see day-to-day.

This technique is named after a [tool](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec) from the Sysinternals Suite and allows system administrators to execute processes on a remote system via the Service Control Manager.

This is the only built-in technique that performs remote injection by default.  It uploads the special service binary payload to disk and creates a new service to run it.  The service binary payload spawns a new temporary process, injects Beacon shellcode into it, and then closes itself.  It does this so that it, along with the corresponding service, can be deleted immediately.

```powershell
beacon> jump psexec64 lon-ws-1 smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337) on lon-ws-1 via Service Control Manager (\\lon-ws-1\ADMIN$\08453ee.exe)

Started service 08453ee on lon-ws-1
[+] established link to child beacon: 10.10.120.10

```

![[Pasted image 20250714102728.png]]

> Beacons executed via PsExec will always run as SYSTEM.