Running processes, and system information
```powershell
tasklist /svc
systeminfo
```

Environment variables
```pwoershell
set 
```
### Key Data Points

`OS name`: Knowing the type of Windows OS (workstation or server) and level (Windows 7 or 10, Server 2008, 2012, 2016, 2019, etc.) will give us an idea of the types of tools that may be available (such as the `PowerShell` version), or lack thereof on legacy systems. This would also identify the operating system version for which there may be public exploits available.

`Version`: As with the OS [version](https://en.wikipedia.org/wiki/Comparison_of_Microsoft_Windows_versions), there may be public exploits that target a vulnerability in a specific version of Windows. Windows system exploits can cause system instability or even a complete crash. Be careful running these against any production system, and make sure you fully understand the exploit and possible ramifications before running one.

`Running Services`: Knowing what services are running on the host is important, especially those running as `NT AUTHORITY\SYSTEM` or an administrator-level account. A misconfigured or vulnerable service running in the context of a privileged account can be an easy win for privilege escalation.

