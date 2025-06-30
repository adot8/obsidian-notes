Beacon itself is a Windows DLL.  If you picture the 'modules' (or DLLs) loaded into a Windows process, you expect to see candidates such as ntdll.dll, kernel32.dll and more.  These DLLs are loaded by the process to access required functionality within them.  To execute a Beacon payload, we need to have its DLL loaded into a process and (usually) kick off a new thread to call its entry point.

![[Pasted image 20250626155513.png]]

Windows DLLs are usually loaded from disk, typically from `%windir%\system32`, but this is not ideal as we don't want to rely on dropping a Beacon DLL to disk before it can be executed.  Instead, Beacon is implemented as a reflective DLL, based on [Stephen Fewer's work](https://github.com/stephenfewer/ReflectiveDLLInjection).  This is a type of DLL that can be loaded into a process purely from memory, rather than from disk.  This requires the DLL to have a reflective loader, which is responsible for re-loading its own image into memory and performing tasks such as loading dependant modules.  The 'overview' section of Stephen's repository goes through the various steps in more detail.

The following illustrates an over-simplified PE structure.

![[Pasted image 20250626155754.png]]