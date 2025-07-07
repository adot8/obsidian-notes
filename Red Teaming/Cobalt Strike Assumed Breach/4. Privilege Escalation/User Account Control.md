
Windows provides a mechanism called Mandatory Integrity Control (MIC) that helps control access to securable objects, such as processes and files.  There are four main integrity levels, creatively called _low_, _medium_, _high_, and _system_.  A security principal cannot modify an object that is assigned with a higher integrity level than their own.

A user, even one that is in the local administrators group on a computer, is only provided with a medium integrity level by default.  You can see this casually by running `whoami /groups` in a command prompt.

![[Pasted image 20250707104754.png]]

From the image above, we can see that the user is a member of the local administrators group but the instance of cmd.exe is only running in a medium-integrity level.  This causes the system-level action of adding a new local user to fail with an access denied error.

User Account Control (UAC) is a defence-in-depth feature of Windows that acts as a gatekeeper to MIC.  To perform an administrative action such as the one above, an administrator user must specifically request to launch cmd.exe in an elevated context (i.e. with a high-integrity level).  UAC provides a prompt for consent or prompt for credentials that the user must accept.

![[Pasted image 20250707104836.png]]

The new instance of cmd.exe will then be running with a high integrity level and the administrative action is permitted.

![[Pasted image 20250707104857.png]]UAC is designed to prevent malware running under the context of an administrative user from silently reading or modifying the system.  However, there are techniques [T1548.002](https://attack.mitre.org/techniques/T1548/002/) by which it can be bypassed.

## Elevators & Exploits

Cobalt Strike provides two built-in primitives for executing code in a high-integrity context, which it calls 'elevators' and 'exploits'.  An _elevator_ runs any arbitrary command (with arguments) via the `runasadmin` command.  An _exploit_ spawns a new Beacon session via the `elevate` command.  There are some UAC bypasses already built into both commands and operators can even add their own via Aggressor.  The implementation of these can be anything that Beacon is capable of executing, including PowerShell, .NET, reflective DLL, and BOF.

## elevate

The syntax for the `elevate` command is `elevate [exploit] [listener]` and executing it by itself will show the available exploits.

```powershell
beacon> elevate

Beacon Local Exploits
=====================

    Exploit                         Description
    -------                         -----------
    cve-2020-0796                   SMBv3 Compression Buffer Overflow (SMBGhost) (CVE 2020-0796)
    ms14-058                        TrackPopupMenu Win32k NULL Pointer Dereference (CVE-2014-4113)
    ms15-051                        Windows ClientCopyImage Win32k Exploit (CVE 2015-1701)
    ms16-016                        mrxdav.sys WebDav Local Privilege Escalation (CVE 2016-0051)
    svc-exe                         Get SYSTEM via an executable run as a service
    uac-schtasks                    Bypass UAC with schtasks.exe (via SilentCleanup)
    uac-token-duplication           Bypass UAC with Token Duplication
```

## runasadmin

The syntax for the `runasadmin` command is `runasadmin [exploit] [command] [args]` and executing it by itself will show the available elevators.

```powershell
beacon> runasadmin

Beacon Command Elevators
========================

    Exploit                         Description
    -------                         -----------
    ms16-032                        Secondary Logon Handle Privilege Escalation (CVE-2016-099)
    uac-cmstplua                    Bypass UAC with CMSTPLUA COM interface
    uac-eventvwr                    Bypass UAC with eventvwr.exe
    uac-schtasks                    Bypass UAC with schtasks.exe (via SilentCleanup)
    uac-token-duplication           Bypass UAC with Token Duplication
    uac-wscript                     Bypass UAC with wscript.exe
```

> The flexibility of `runasadmin` means that you can execute any arbitrary command.