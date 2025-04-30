In terms of **OPSEC**, avoid BloodHound lights up SIEMS like a Christmas tree because of the large amount of data being collected in a short amount of time.

#### Legacy
```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\BloodHound-
master\BloodHound-master\Collectors\SharpHound.exe -args --
collectionmethods All --zipfilename shout
```
#### CE
```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -
args --collectionmethods All --zipfilename shout
```
#### BloodHound OPSEC
-  To make BloodHound collection stealthy, remove noisy collection methods like RDP, DCOM, PSRemote and LocalAdmin.
- Use the `-ExcludeDCsto` avoid detection by MDI:

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\SharpHound\SharpHound.exe -args --collectionmethods Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets,CertServices --excludedcs --zipfilename shout
```

> [!NOTE] **Note**
> Remember to remove the `CertServices` collection method when using BloodHound legacy collector.

#### SOAPHound OPSEC
- Use SOAPHound for even **more stealth**. Far less quieter than BloodHound
- It talks to Active Driectory Web Services (`ADWS` - Port 9389) in place of sending `LDAP` queries - just like the AD Module
	- Almost no network-based detection (like MDI).
	- it retrieves information about all objects (objectGuid=) and then process them. It means limited LDAP queries - less chance of endpoint detection.

Build a cache that includes basic info about domain objects
```powershell
SOAPHound.exe --buildcache -c C:\AD\Tools\cache.txt
```

Collect BloodHound compatible data
```powershell
SOAPHound.exe -c C:\AD\Tools\cache.txt --bhdump -o C:\AD\Tools\bloodhound-output --nolaps
```