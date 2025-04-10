### Domain Enum
```powershell
Get-Domain
Get-Domain -Domain moneycorp.local
Get-DomainSID
```

```powershell
Get-DomainPolicyData
```

```powershell
Get-DomainController
Get-DomainController -Domain moneycorp.local
```

> [!NOTE] **Note**
> Policy data will be important for password policies and Kerberos ticket information (**needed for forging tickets, must match for OPSEC**)

### Users Enum
```powershell
Get-DomainUser | select samaccountname
Get-DomainUser -Identity student1
```

> [!NOTE] **IMPORTANT**
> Properties of users are important. Check login counts to see who might be a **HONEYPOT!!**
> 
> 	- Less than 5-10 logon counts
> 	- Last logon time
> 	-  Bad password time (should be wrong at least once)

```powershell
Get-DomainUser -Identity student1 -Properties *
Get-DomainUser -Properties samaccountname,logonCount,Description
```

Grep out a specific string
```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
```

### Computer Enum
> [!NOTE] **Note**
> Computer objects and Computers are different things
> Check the logon count to find out
> ```powershell
> Get-DomainComputer | select cn,logoncount
> ```

```powershell
Get-DomainComputer | select cn
Get-DomainComputer | select -ExpandProperty dnshostname
```

```powershell
Get-DomainComputer | select cn
Get-DomainComputer | select -ExpandProperty dnshostname
```

### Group enum
```powershell
Get-DomainGroup | select Name
Get-DomainGroup -Domain <targetdomain>
```

> [!NOTE] **IMPORTANT**
> Need to specify the domain for Enterprise Admins and others to show
> ```powershell
> Get-DomainGroup -Name *admin* -Domain [domain] -properties name
> ```

```powershell
Get-DomainGroup *admin*
Get-DomainGroup -Name *admin* | select cn
```

### Domain Group Membership

> [!NOTE] **NOTE**
> Helpful to rename the local machine **Administrators** for post enumeration
> 
> SID will be the same for domain Admin
> 
> **ALSO** having a target user in multiple groups helps with privileges to other objects down the road

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
Get-DomainGroupMember -Identity "Domain Admins" -Recurse | select MemberName
Get-DomainGroupMember -Identity "Enterprise Admins" -Recurse -domain <domain>
```

```powershell
Get-DomainGroup -UserName "student1"
Get-DomainGroup -UserName "student1" | select name
```

### Local Group Membership
> [!NOTE] **NOTE**
> You can view local groups on Domain Controllers but need local Admin on other remote computers to list them

```powershell
Get-NetLocalGroup -ComputerName dcorp-dc
Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators
```

### Shares, Sensitive files and FileServers
```powershell
Invoke-ShareFinder -Verbose
Invoke-FileFinder -Verbose
Get-NetFileServer
```