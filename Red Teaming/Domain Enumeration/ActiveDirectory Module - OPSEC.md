[ADModule](https://github.com/samratashok/ADModule)
[Windows ADModule](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps)
### Domain Enum
```powershell
Import-Module .\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module .\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

```powershell
Get-ADDomain
Get-ADDomain -Identity moneycorp.local
(Get-ADDomain).DomainSID
```

> [!NOTE] **NOTE**
> Policy data will be important for password policies and Kerberos ticket information
> The Ticket must be compatible with the Kerberos Policy 
> **(needed for forging tickets, must match for OPSEC)**

```powershell
(Get-DomainPolicyData).systemaccess
(Get-DomainPolicyData -domain moneycorp.local).systemaccess
```

```powershell
Get-ADDomainController
Get-ADDomainController -DomainName moneycorp.local -Discover
```

### Users Enum
**ONLY TARGET ACTIVE USERS**

```powershell
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity student1 -Properties *
 Get-ADUser -Filter * -Properties * | select CN
```

> [!NOTE] **IMPORTANT**
> Properties of users are important. Check login counts to see who might be a **HONEYPOT!!**
> 
> 	- Less than 5-10 logon counts
> 	- Last logon time
> 	-  Bad password time (should be wrong at least once)

```powershell
Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name

Get-ADUser -Filter * -Properties * | select name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```

Grep out a specific string
```powershell
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description
```

### Computer Enum
> [!NOTE] **Note**
> Computer objects and Computers are different things
> Check the logon count to find out
> ```powershell
> Get-ADComputer -Filter * -Properties * | select Name,logoncount
> ```

```powershell
Get-ADComputer -Filter * | select Name
Get-ADComputer -Filter * -Properties *
Get-ADComputer -Filter 'OperatingSystem -like "*Server 2022*"' -
Properties OperatingSystem | select Name,OperatingSystem

Get-ADComputer -Filter * -Properties DNSHostName | %{TestConnection 
-Count 1 -ComputerName $_.DNSHostName}
```

### Group Enum
```powershell
Get-ADGroup -Filter * | select Name
Get-ADGroup -Filter * -Properties *
```

> [!NOTE] **Note**
> Need to specify the domain for Enterprise Admins and others to show
> ```powershell
> Get-ADGroup -Filter 'Name -like "*admin*"' -Domain [domain] | select Name 
> ```

```powershell
Get-ADGroup -Filter 'Name -like "*admin*"' | select Name 
```

### Domain Group Membership
> [!NOTE] **NOTE**
> Helpful to rename the local machine Administrators for post enumeration
> 
> SID will be the same for domain Admin
> 
> **ALSO** having a target user in multiple groups helps with privileges to other objects down the road

```powershell
Get-ADGroupMember -Identity "Domain Admins" -Recursive 
Get-ADPrincipalGroupMembership -Identity student1
```

### Organizational Units
```powershell
Get-ADOrganizationalUnit -Filter * -Properties *
```