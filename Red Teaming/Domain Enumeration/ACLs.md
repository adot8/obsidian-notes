View ACLs associated with an object
```powershell
Get-DomainObjectAcl -SamAccountName student1 -ResolveGUIDs
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs

$sid = Convert-NameToSid student548
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

View ACLs associated with the specified prefix to be used for search
```powershell
Get-DomainObjectAcl -SearchBase "LDAP://CN=RDP Users,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs -Verbose
```

The way to read the output would be:

- On `ObjectDN`: Domain Admins
- The `SecurityIdentifier` (SID / object)
- Has `ActiveDirectoryRights`: GenricAll (All rights)
![[Pasted image 20250429064219.png]]

Search for Write, Modify and GenericAll ACEs
```powershell
Find-InterestingDomainAcl -ResolveGUIDs
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "student548"}
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

Specified Path
```powershell
Get-PathAcl -Path "\\dcorp-dc.dollarcorp.moneycorp.local\sysvol"
```