
Constrained delegation is set on a front-end service and defines the back-end service(s) to which it can delegate to.  Modifying the msDS-AllowedToDelegateTo attribute of a service account (user or computer) requires the [SeEnableDelegationPrivilege](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/enable-computer-and-user-accounts-to-be-trusted-for-delegation) user-right assignment on domain controllers, which is only assigned to enterprise and domain admins.  Microsoft saw this as a problem because the service administrators had no useful way of knowing which front-end services delegated to resources that they were responsible for.

Windows 2012 introduced yet another form of delegation, called resource-based constrained delegation (often shorted to RBCD), which puts delegation control into the hands of the service administrators (e.g. it no longer requires SeEnableDelegationPrivilege to configure).  Now, instead of a front-end service dictating to which back-end services it can delegate to; it's the back-end service that controls which front-end services can delegate to it.

This is controlled by defining the front-end service account(s) in the `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute of the service account running the back-end service.  The only privilege required is write access to this attribute, which will typically be granted to an appropriate domain group via the Delegation of Control Wizard.

![[Pasted image 20250722105219.png]]

An admin with these delegated rights can then add the desired RBCD entries using the RSAT PowerShell cmdlets.

```powershell
$front = Get-ADComputer -Identity 'lon-ws-1'
$back = Get-ADComputer -Identity 'lon-fs-1'

Set-ADComputer -Identity $back -PrincipalsAllowedToDelegateToAccount $front
```