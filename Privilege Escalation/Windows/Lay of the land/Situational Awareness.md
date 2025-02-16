Networking awareness
```powershell
ipconfig /all
arp -a
route print
```

AV/EDR awareness (Defender settings, AppLocker rules, test AppLocker policy)
```powershell
Get-MpComputerStatus
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```