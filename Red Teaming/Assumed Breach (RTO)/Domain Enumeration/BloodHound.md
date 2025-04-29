In terms of **OPSEC**, avoid BloodHound lights up SIEMS like a Christmas tree because of the large amount of data being collected in a short amount of time.

### Legacy
```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-
master\BloodHound-master\Collectors\SharpHound.exe -args --
collectionmethods All
```

### CE
```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -
args --collectionmethods All
```

### OPSEC
-  To make BloodHound collection stealthy, remove noisy collection methods like RDP, DCOM, PSRemote and LocalAdmin.
- Use the `-ExcludeDCsto` avoid detection by MDI:

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SharpHound\SharpHound.exe -
args --collectionmethods
Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTarg
ets,CertServices --excludedcs
```

> [!NOTE] **Note**
> Remember to remove the `CertServices` collection method when using BloodHound legacy collector.
