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