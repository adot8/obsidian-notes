### Find local admin access on machines for current user
![[Pasted image 20250501061333.png]]
```powershell
Find-LocalAdminAccess -Verbose
```

> [!NOTE] **IMPORTANT**
> **THIS IS VERY NOISY DUE TO THE HIGH LOGIN AND LOGOFF LOG TRAFFIC**
> 
> Run it in batches of machines instead of all the machines at once

We can also use the scripts `Find-PSRemotingLocalAdminAccess.ps1` and `Find-WMILocalAdminAccess.ps1`

```powershell
. .\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess.ps1
```

### Find domain admin sessions ([SessionHunter](https://github.com/Leo4j/Invoke-SessionHunter))
We can dump their creds if we have local admin
```powershell
Invoke-SessionHunter -FailSafe
```

###### OPSEC OPTION
```powershell
Invoke-SessionHunter -NoPortScan -Targets C:\AD\Tools\servers.txt
```

###### PowerView option
```powershell
Find-DomainUserLocation -Verbose
Find-DomainUserLocation -CheckAccess
Find-DomainUserLocation -UserGroupIdentity "RDPUsers"
```

> [!NOTE] **NOTE**
> `Server 2019` and onwards, local administrator privileges are required to list sessions
