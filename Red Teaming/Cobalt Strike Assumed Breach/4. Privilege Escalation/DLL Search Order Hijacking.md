All but the most basic of applications will reference functions located in external modules, such as DLLs (dynamic-link libraries).  DLLs are PE (portable executable) files that contain common functionalities that can be used by any program.  Windows provides multiple DLLs that can be used by programmers, such as kernel32.dll, user32.dll, etc, to interact with various parts of Windows.  Developers can also write their own DLLs for use in their applications.

When referencing a DLL, it's atypical to provide its full path because it cannot be reliably known ahead of time.  For example, specifying `C:\Windows\System32\kernel32.dll` will not work in all cases because Windows does not have to be installed on a drive labelled C.  Specifying something like `C:\Program Files\My Application\MyDll.dll` will also not work in all cases because a user may choose to install the application in a different location.

Therefore, a program usually only references external modules by their name, e.g. "kernel32" or "MyDll", and Windows becomes responsible for finding its location on disk.  This is called the [DLL search order](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-search-order).

Although there is some nuance depending on the type of application, the search order for most will be:

- The executing directory.
    
- The System32 directory.
    
- The 16-bit System directory.
    
- The Windows directory.
    
- The current working directory of the program.
    
- Directories in the PATH environment variable.
    

DLL search order hijacking is a technique [T1574.001](https://attack.mitre.org/versions/v16/techniques/T1574/001/) where an adversary drops a malicious module of the same name in a directory that is higher in the search hierarchy than that of the legitimate module.  If the program loading the module is running with elevated privileges, then this will result in an elevation of privilege for the adversary.

Consider this simple example, where a DLL is loaded and freed on a loop.
![[Pasted image 20250707103705.png]]

The path passed to LoadLibrary is relative.  However, there is no _BadDll.dll_ file in the directory the service executable is running from.

```powershell
beacon> ls C:\Program Files\Bad Windows Service\Service Executable

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
 9kb      fil     01/06/2025 16:10:12   BadWindowsService.exe
```