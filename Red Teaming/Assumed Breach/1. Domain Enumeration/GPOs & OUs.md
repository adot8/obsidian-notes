You can only list the GPO's and not their settings
```powershell
Get-DomainGPO 
Get-DomainGPO | select displayname
Get-DomainGPO -ComputerIdentity dcorp-student1
```


> [!NOTE] **NOTE**
> Restricted Groups used to add domain groups to local groups
> If you can compromise a user in the Restricted Groups, it may be a local admin on all the workstations

Get GPO(s) which use Restricted Groups or `groups.xml`for interesting
user. These groups have interesting users
```powershell
Get-DomainGPOLocalGroup
```

**Having write access to a GPO can allow us to create a logon script for all users**

Get users which are in a local group of a machine using GPO
```powershell
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
```

Get machines where the given user is member of a specific group
```powershell
Get-DomainGPOUserLocalGroupMapping -Identity student1 -Verbose
```

#### Organizational Units
```bash
Get-DomainOU
Get-DomainOU -properties name
```

This is the GPO that applies to the OU [gplink]
![[Pasted image 20250430061201.png]]

Get GPO applied on an OU. Read GPOname from gplink attribute from Get-NetOU
```powershell
Get-DomainGPO -Identity "{7478F170-6A0C-490C-B355-9E4618BC785D}"
```

#### Find all computers in an OU
```powershell
(Get-DomainOU -identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```