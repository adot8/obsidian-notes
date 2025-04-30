You can only list the GPO's and not their settings
```powershell
Get-DomainGPO
Get-DomainGPO -ComputerIdentity dcorp-student1
```

List GPO's that apply Restricted Groups which will have interesting users

Get users which are in a local group of a machine using GPO
```powershell
Get-DomainGPOLocalGroup
```

> [!NOTE] **NOTE**
> If you can compromise a user in that group, it may be a local admin on all the workstations

Get machines where the given user is member of a specific group
```powershell
Get-DomainGPOComputerLocalGroupMapping -ComputerIdentity dcorp-student1
```

### Organizational Units