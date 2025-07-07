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