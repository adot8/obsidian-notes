
DLL hijacking is a technique used to force a legitimate application into loading a malicious DLL.  This abuses the Windows DLL search order when an application attempts to load its dependencies.  A simplified view of this load order is:

- The directory the application is in.
    
- The system directory, typically `C:\Windows\System32`.
    
- The 16-bit system directory, typically `C:\Windows\System`.
    
- The Windows directory, typically `C:\Windows`.
    
- The current working directory.
    
- The directories that are listed in the `PATH` environment variable.

A common methodology for finding these hijack opportunities is to run [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) with a filter where:

- The path ends in **.dll**.
    
- The result is **NAME NOT FOUND**.
    

One scenario where these hijacks become possible is when an application attempts to load a DLL that doesn't exist on the system.  An adversary can drop a malicious DLL of the correct filename into a location known to be in the search order that is also writeable by standard users.  Then when the application runs, it will traverse the search order until it finds and loads that DLL.

Finding vulnerable applications that are built into the Windows OS is not as common as it once was, which is where DLL side-loading comes into play.  The Windows Component Store is located in `C:\Windows\WinSxS`.  Although it serves multiple purposes today, its original role was only to support side-by-side assemblies (hence the name SxS).  A 'WinSxS manifest' describes the dependencies and their versions that are required by an application.  WinSxS allows multiple versions of the same dependencies to exist on a system, which reduces the chance of conflicts between applications that are dependant on different versions of the same library.  Old versions of these dependencies get stored in this directory during a Windows update, so vulnerable versions of applications may exist here even though they've been patched in the main OS.

Let's have a look at `ngentask.exe` as an example.  If we run the version currently installed in `C:\Windows\Microsoft.NET\Framework64`, we can see that even though it's looking for some DLLs that cannot be found, it's only searching its directory and the Global Assembly Cache (GAC).

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/2aa39b073378cbfceb298550b57f9ff1.png)

Now, let's find an older version of ngentask in SxS:
```powershell
PS C:\Users\Attacker> ls -Path C:\Windows\WinSxS -Recurse -Filter ngentask.exe | Select -expand FullName

C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.0_none_d4039dd5692796db\ngentask.exe
C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.285_none_cbd46ba524299690\ngentask.exe
C:\Windows\WinSxS\amd64_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15840.3_none_d3fea185692c10e1\ngentask.exe
C:\Windows\WinSxS\x86_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.0_none_1bb0d4ac7da3bfe1\ngentask.exe
C:\Windows\WinSxS\x86_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15805.285_none_1381a27c38a5bf96\ngentask.exe
C:\Windows\WinSxS\x86_netfx4-ngentask_exe_b03f5f7f11d50a3a_4.0.15840.3_none_1babd85c7da839e7\ngentask.exe
```

We can execute these an observe the difference in DLL loading behaviour in ProcMon.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/874152b106730b5e2875f399d6d2809f.png)

Here we can see that it's attempting to load `mscorsvc.dll` by following the standard search order.  The absolute easiest one to hijack is the current working directory, which is just `C:\Users\Attacker` in this case.  If we drop a DLL call mscorsvc.dll into this directory and run this version of ngentask again, it will load the DLL.  You can write your own test DLL or use one that someone has already published, such as [this one](https://github.com/FuzzySecurity/DLL-Template) by [Ruben Boonen](https://x.com/FuzzySec).

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/cd98c93558ce95bf509798b2455b0a0e.png)
