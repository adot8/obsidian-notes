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
> Policy data will be important for password policies and Kerberos ticket information
> > The Ticket must be compatible with the Kerberos Policy 
> **(needed for forging tickets, must match for OPSEC)**

### Users Enum
**ONLY TARGET ACTIVE USERS**

```powershell
Get-DomainUser | select samaccountname
Get-DomainUser -Identity student1
```

> [!NOTE] **IMPORTANT**
> Properties of users are important. Check login counts to see who might be a **HONEYPOT!!**
> 
> 	- Less than 5-10 logon counts
> 	- Last logon time
> 	- Bad password time (should be wrong at least once)

```powershell
Get-DomainUser -Identity student1 -Properties *
Get-DomainUser | select samaccountname,logonCount,Description
```

Grep out a specific string
```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description
```

### Computer Enum
> [!NOTE] **Note**
> Domain users can add up to 10 computer objects
> Computer objects and Computers are different things
> Check the logon count to find out
> ```powershell
> Get-DomainComputer | select cn,logoncount
> ```

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
> **Need to specify the domain/forest root for Enterprise Admins and others to show**
> ```powershell
> Get-DomainGroup -Name *admin* -Domain [domain] -properties name
> ```

```powershell
Get-DomainGroup *admin*
Get-DomainGroup -Name *admin* -Domain moneycorp.local| select name, objectsid, description
```

### Domain Group Membership

> [!NOTE] **NOTE**
> Helpful to rename the local machine **Administrators** for post enumeration
> 
> SID will be the same for domain Admin
> 
> **ALSO** having a target user in multiple groups helps with privileges to other objects down the road

**STAY AWAY FROM DOMAIN ADMINS, THEY ARE THE MOST MONITORED. FOCUS ON THE OBJECTIVES. NOT RUSHING TO GET DOMAIN ADMIN IS THE DIFFERENCE BETWEEN OPSEC AND CTFS**

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
Get-DomainGroupMember -Identity "Domain Admins" -Recurse | select MemberName
Get-DomainGroupMember -Identity "Enterprise Admins" -Recurse -domain <domain>
```

```powershell
Get-DomainGroup -UserName "student1"
Get-DomainGroup -UserName "student1" | select name
```

The difference between` Domain Admins `and `Enterprise Admins` is that `Enterprise Admins` have **direct access** to the other domain controllers in the network/forest because they sit at the forest level. However, `Domain Admins` sit at the domain level so there is not any **direct access** to the other domain controllers, compromise of them can be quick but the **direct access** isn't there.

**Regardless, if a single domain is compromised, the entire forest is compromised**
### Local Group Membership
> [!NOTE] **NOTE**
> You can view local groups on Domain Controllers but need local Admin on other remote computers to list them

```powershell
Get-NetLocalGroup -ComputerName dcorp-dc
Get-NetLocalGroupMember -ComputerName dcorp-dc -GroupName Administrators
```

### Shares, Sensitive files and FileServers
Use [PowerHuntShares](https://github.com/NetSPI/PowerHuntShares) It can discover shares, sensitive files, ACLs for shares, networks,
computers, identities etc. and generates a nice HTML report.

```powershell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\AD\Tools -HostList C:\AD\Tools\servers.txt
```

The `servers.txt` in the above command does not include the **domain controller** for better **OPSEC**. DO NOT SCAN MORE THAN 15 MACHINES AT A TIME.

```powershell
Invoke-ShareFinder -Verbose
Invoke-FileFinder -Verbose
Get-NetFileServer
```

