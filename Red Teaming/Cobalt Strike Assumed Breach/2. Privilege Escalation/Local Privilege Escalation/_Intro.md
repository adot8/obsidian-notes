It's a common scenario, particularly immediately after initial access, for adversaries to be limited to standard user level access to a compromised system.  Privilege escalation is a tactic [[TA0004](https://attack.mitre.org/tactics/TA0004/)] by which they attempt to exploit a vulnerability or misconfiguration to gain additional privileges.  This is often a necessary prerequisite to carrying out additional tactics, such as [credential access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b78a7d9f99e969c066ebf).

## Windows Accounts

There are three main account groups which are relevant to privilege escalation.

### Users

These are low-privilege accounts that don't have permissions to access or modify anything outside of their own files.

### Administrators

Members of the local Administrators group are intended to be used for system administration.  As such, they have Full Control permissions over most objects on the computer, such as directories, files, and services.

### Service User Accounts

These accounts are built into Windows and are used by the Service Control Manager (SCM) to run many of the services that Windows relies on.  These are:

- #### LocalService
    
    This account, often written as **LOCAL SERVICE**, has minimum privileges on the local computer and authenticates on the network using anonymous credentials.
    
- #### NetworkService
    
    This account, often written as **NETWORK SERVICE**, has minimum privileges on the local computer but authenticates on the network as the computer by presenting the computer's credentials.  
    
- #### LocalSystem
    
    This account, often written as **SYSTEM**, has Full Control over most system objects like the local Administrators group does.  In some cases, it has privileges that Administrators do not, so is generally considered to be the highest level of access you can obtain on Windows.
    

These accounts are not recognised by the security subsystem, so they don't appear when running `net user` for example.  Their privileges come from [User Rights Assignments](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/user-rights-assignment), which are security policy settings in Windows.

## User Rights Assignment

These settings allow for more fine-grained control over an account's privileges.  System administrators may require accounts to run services that are more privileged than standard users, but less-so than full administrators.  The service user accounts listed above inherit their privileges from user right assignment, rather than local group membership.  User right assignments are split into two camps:

- #### Logon Rights
    
    These control how and where an account is authorised to log into a computer.  For example, _SeBatchLogonRight_ allows a user to be logged in by a batch-queue service such as the Task Scheduler.
    
- #### User Rights
    
    These control access to objects and can even override explicit permissions on them.  For example, _SeBackupPrivilege_ allows a user to read any file, even if they haven't been granted read access to it.
    

There are some sensitive user rights that can be abused to obtain SYSTEM privileges, like _SeImpersonatePrivilege_.  This can be held by accounts running services such as web or SQL servers.

## Vulnerable Services

Services are one of the largest attack surfaces for privilege escalation on Windows because they frequently run as SYSTEM and can be misconfigured or vulnerable in a variety of ways, including:

- DLL Search Order Hijacking ([T1574.001](https://attack.mitre.org/techniques/T1574/001/)).
- Path Interception by:  
    - PATH Environment Variable [T1574.007](https://attack.mitre.org/techniques/T1574/007/)
    - Search Order Hijacking [T1574.008](https://attack.mitre.org/techniques/T1574/008/)
    - Unquoted Path [T1574.009](https://attack.mitre.org/techniques/T1574/009/)
- Weak Service File Permissions [T1574.010](https://attack.mitre.org/techniques/T1574/010/)
- Weak Service Registry Permissions [T1574.011](https://attack.mitre.org/techniques/T1574/011/)
- Generic software vulnerabilities [T1068](https://attack.mitre.org/techniques/T1068/)

```powershell
beacon> sc_enum

SERVICE_NAME: BadWindowsService
DISPLAY_NAME: BadWindowsService
	TYPE                           : 16 WIN32_OWN
	STATE                          : 4 RUNNING
	WIN32_EXIT_CODE                : 0
	SERVICE_EXIT_CODE              : 0
	CHECKPOINT                     : 0
	WAIT_HINT                      : 0
	PID                            : 3004
	FLAGS                          : 0
	TYPE                           : 10 WIN32_OWN
	START_TYPE                     : 2 AUTO_START
	ERROR_CONTROL                  : 1 NORMAL
	BINARY_PATH_NAME               : C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe
	LOAD_ORDER_GROUP               : 
	TAG                            : 0
	DISPLAY_NAME                   : BadWindowsService
	DEPENDENCIES                   : 
	SERVICE_START_NAME             : LocalSystem
	RESET_PERIOD (in seconds)      : 0
	REBOOT_MESSAGE                 : 
	COMMAND_LINE                   :
```

There are also multiple automated tools that can help in finding these vulnerabilities, including [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) and [SharpUp](https://github.com/GhostPack/SharpUp).  However, it's often useful to do your own analysis, particularly when looking for vulnerabilities within the applications themselves.  Native applications can be reversed using tools such as [Ghidra](https://github.com/NationalSecurityAgency/ghidra) or [IDA Free](https://hex-rays.com/ida-free); and .NET applications with [dotPeek](https://www.jetbrains.com/decompiler/) or [dnSpy](https://github.com/dnSpy/dnSpy).