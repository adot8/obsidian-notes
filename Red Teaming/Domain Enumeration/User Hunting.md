### Find local admin access on machines for current user

```powershell
Find-LocalAdminAccess -Verbose
```

> [!NOTE] **IMPORTANT**
> **THIS IS VERY NOISY DUE TO THE DOMAIN CONTROLLER BEING THE FIRST COMPUTER ON THE LIST BY DEFAULT (RAT) AND THE HIGH LOGIN AND LOGOFF LOG TRAFFIC**
> 
>  Remove the DC from the computer list and run it in batches of machines instead of all the machines at once

We can also use the scripts `Find-PSRemotingLocalAdminAccess.ps1` and `Find-WMILocalAdminAccess.ps1`

```powershell
. .\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess.ps1
```

### Find domain admin sessions ([SessionHunter](https://github.com/Leo4j/Invoke-SessionHunter))
We can dump their creds if we have local admin
```powershell
Invoke-SessionHunter -FailSafe
Invoke-SessionHunter -CheckAsAdmin
Invoke-SessionHunter -CheckAsAdmin -UserName "ferrari\Administrator" -Password "P@ssw0rd!"
```

###### OPSEC OPTION
```powershell
Invoke-SessionHunter -NoPortScan -Targets C:\AD\Tools\servers.txt
```

###### PowerView option
Find computers where a domain admin session is available and current user
has admin access 
```powershell
Find-DomainUserLocation -Verbose
Find-DomainUserLocation -CheckAccess
Find-DomainUserLocation -UserGroupIdentity "RDPUsers"
```

Find computers (File Servers and Distributed File servers) where a domain
admin session is available
```powershell
Find-DomainUserLocation -Stealth
```

> [!NOTE] **NOTE**
> `Server 2019` and onwards, local administrator privileges are required to list sessions
