### Cmdkey Saved Credentials
#### Listing Saved Credentials
The [cmdkey](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) command can be used to create, list, and delete stored usernames and passwords. Users may wish to store credentials for a specific host or use it to store credentials for terminal services connections to connect to a remote host using Remote Desktop without needing to enter a password

```powershell
C:\htb> cmdkey /list

    Target: LegacyGeneric:target=TERMSRV/SQL01
    Type: Generic
    User: inlanefreight\bob
```

When we attempt to RDP to the host, the saved credentials will be used.

![image](https://academy.hackthebox.com/storage/modules/67/cmdkey_rdp.png)

We can also attempt to reuse the credentials using `runas` to send ourselves a reverse shell as that user, run a binary, or launch a PowerShell or CMD console with a command such as:
 
Run Commands as Another User
```powershell
PS C:\htb> runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

### Retrieving Saved Credentials from Chrome

Users often store credentials in their browsers for applications that they frequently visit. We can use a tool such as [SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome.

```powershell
PS C:\htb> .\SharpChrome.exe logins /unprotect
```

### Password Managers
Many companies provide password managers to their users. This may be in the form of a desktop application such as `KeePass`, a cloud-based solution such as `1Password`, or an enterprise password vault such as `Thycotic` or `CyberArk`. Gaining access to a password manager, especially one utilized by a member of the IT staff or an entire department, may lead to administrator-level access to high-value targets such as network devices, servers, databases, etc. We may gain access to a password vault through password reuse or guessing a weak/common password. 

Some password managers such as `KeePass` are stored locally on the host. If we find a `.kdbx` file on a server, workstation, or file share, we know we are dealing with a `KeePass` database which is often protected by just a master password. If we can download a `.kdbx` file to our attacking host, we can use a tool such as [keepass2john](https://gist.githubusercontent.com/HarmJ0y/116fa1b559372804877e604d7d367bbc/raw/c0c6f45ad89310e61ec0363a69913e966fe17633/keepass2john.py) to extract the password hash

```bash
keepass2john ILFREIGHT_Help_Desk.kdbx 
hashcat -m 13400 keepass_hash ~/rockyou.txt -O
```

### Email
If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox, we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper).

```powershell
Invoke-SelfSearch -Mailbox current-user@domain.com
```

### SessionGopher
We can use [SessionGopher](https://github.com/Arvanaghi/SessionGopher) to extract saved PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP credentials. The tool is written in PowerShell and searches for and decrypts saved login information for remote access tools. It can be run locally or remotely. It searches the `HKEY_USERS` hive for all users who have logged into a domain-joined (or standalone) host and searches for and decrypts any saved session information it can find. It can also be run to search drives for PuTTY private key files (.ppk), Remote Desktop (.rdp), and RSA (.sdtid) files.

We need local admin access to retrieve stored session information for every user in HKEY_USERS, but it is always worth running as our current user to see if we can find any useful credentials
```powershell
Import-Module .\SessionGopher.ps1
Invoke-SessionGopher -Target WINLPE-SRV01
```

### Registry
AutoLogon
```powershell
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
```

Putty
```powershell
reg query "HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions"
reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```

Wifi
```powershell
netsh wlan show profile ilfreight_corp key=clear
```