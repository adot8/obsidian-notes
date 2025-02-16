#### Running processes, and system information
It is essential to become familiar with standard Windows processes such as [Session Manager Subsystem (smss.exe)](https://en.wikipedia.org/wiki/Session_Manager_Subsystem), [Client Server Runtime Subsystem (csrss.exe)](https://en.wikipedia.org/wiki/Client/Server_Runtime_Subsystem), [WinLogon (winlogon.exe)](https://en.wikipedia.org/wiki/Winlogon), [Local Security Authority Subsystem Service (LSASS)](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service), and [Service Host (svchost.exe)](https://en.wikipedia.org/wiki/Svchost.exe), among others and the services associated with them. Being able to spot standard processes/services quickly will help speed up our enumeration and enable us to hone in on non-standard processes/services, which may open up a privilege escalation path. In the example above, we would be most interested in the `FileZilla` FTP server running and would attempt to enumerate the version to look for public vulnerabilities or misconfigurations such as FTP anonymous access, which could lead to sensitive data exposure or more.

The `systeminfo` command will show if the box has been patched recently and if it is a VM. If the box has not been patched recently, getting administrator-level access may be as simple as running a known exploit. Google the KBs installed under [HotFixes](https://www.catalog.update.microsoft.com/Search.aspx?q=hotfix) to get an idea of when the box has been patched. This information isn't always present, as it is possible to hide hotfixes software from non-administrators. The `System Boot Time` and `OS Version` can also be checked to get an idea of the patch level. If the box has not been restarted in over six months, chances are it is also not being patched.
```powershell
tasklist /svc
systeminfo
wmic qfe
Get-HotFix | ft -AutoSize
```

#### Environment variables
The environment variables explain a lot about the host configuration. To get a printout of them, Windows provides the `set` command. One of the most overlooked variables is `PATH`. In the output below, nothing is out of the ordinary. However, it is not uncommon to find administrators (or applications) modify the `PATH`. One common example is to place Python or Java in the path, which would allow the execution of Python or . JAR files. If the folder placed in the PATH is writable by your user, it may be possible to perform DLL Injections against other applications. Remember, when running a program, Windows looks for that program in the CWD (Current Working Directory) first, then from the PATH going left to right. This means if the custom path is placed on the left (before C:\Windows\System32), it is much more dangerous than on the right.
```pwoershell
set 
```

#### Installed Programs
WMI can also be used to display installed software. This information can often guide us towards hard-to-find exploits. Is `FileZilla`/`Putty`/etc installed? Run `LaZagne` to check if stored credentials for those applications are installed. Also, some programs may be installed and running as a service that is vulnerable.
```powershell
wmic product get name
Get-WmiObject -Class Win32_Product |  select Name, Version
```

#### Display Running Processes

The [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command will display active TCP and UDP connections which will give us a better idea of what services are listening on which port(s) both locally and accessible to the outside. We may find a vulnerable service only accessible to the local host (when logged on to the host) that we can exploit to escalate privileges.

```powershell
netstat -ano
```

#### Logged-In Users

It is always important to determine what users are logged into a system. Are they idle or active? Can we determine what they are working on? While more challenging to pull off, we can sometimes attack users directly to escalate privileges or gain further access. During an evasive engagement, we would need to tread lightly on a host with other user(s) actively working on it to avoid detection.

```powershell
query user
```

#### Current user and groups
```powershell
echo %USERNAME%
whoami /priv
whoami /groups
```

#### All users and groups
```powershell
net user
net localgroup
net localgroup administrators
```

#### Password Policy
```bash
net accounts
```
### Key Data Points

`OS name`: Knowing the type of Windows OS (workstation or server) and level (Windows 7 or 10, Server 2008, 2012, 2016, 2019, etc.) will give us an idea of the types of tools that may be available (such as the `PowerShell` version), or lack thereof on legacy systems. This would also identify the operating system version for which there may be public exploits available.

`Version`: As with the OS [version](https://en.wikipedia.org/wiki/Comparison_of_Microsoft_Windows_versions), there may be public exploits that target a vulnerability in a specific version of Windows. Windows system exploits can cause system instability or even a complete crash. Be careful running these against any production system, and make sure you fully understand the exploit and possible ramifications before running one.

`Running Services`: Knowing what services are running on the host is important, especially those running as `NT AUTHORITY\SYSTEM` or an administrator-level account. A misconfigured or vulnerable service running in the context of a privileged account can be an easy win for privilege escalation.
