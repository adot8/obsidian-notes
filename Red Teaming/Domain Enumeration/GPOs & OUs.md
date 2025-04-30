You can only list the GPO's and not their settings
```powershell
Get-DomainGPO 
Get-DomainGPO | select displayname
Get-DomainGPO -ComputerIdentity dcorp-student1
```

List GPO's that apply Restricted Groups which will have interesting users

Get users which are in a local group of a machine using GPO
```powershell
Get-DomainGPOLocalGroup
```

> [!NOTE] **NOTE**
> If you can compromise a user in the Restricted Groups, it may be a local admin on all the workstations
> 

**Having write access to a GPO can allow us to create a logon script for all users**

Get machines where the given user is member of a specific group
```powershell
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
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
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}"
```

#### Find all computers in an OU
```powershell
(Get-DomainOU -identity <OU>).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```