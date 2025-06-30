Beacon itself is a Windows DLL.  If you picture the 'modules' (or DLLs) loaded into a Windows process, you expect to see candidates such as ntdll.dll, kernel32.dll and more.  These DLLs are loaded by the process to access required functionality within them.  To execute a Beacon payload, we need to have its DLL loaded into a process and (usually) kick off a new thread to call its entry point.

![[Pasted image 20250626155513.png]]

Windows DLLs are usually loaded from disk, typically from `%windir%\system32`, but this is not ideal as we don't want to rely on dropping a Beacon DLL to disk before it can be executed.  Instead, Beacon is implemented as a reflective DLL, based on [Stephen Fewer's work](https://github.com/stephenfewer/ReflectiveDLLInjection).  This is a type of DLL that can be loaded into a process purely from memory, rather than from disk.  This requires the DLL to have a reflective loader, which is responsible for re-loading its own image into memory and performing tasks such as loading dependant modules.  The 'overview' section of Stephen's repository goes through the various steps in more detail.

The following illustrates an over-simplified PE structure.

![[Pasted image 20250626155754.png]]

Beacon's DLL exports a function called **ReflectiveLoader**, which when called, walks back over its own image, maps a new copy of itself into memory, and calls its entry point.  Because the DLL is exported as shellcode, another small shellcode stub is written over the DOS Header.  This shellcode is responsible for jumping code execution to the exported ReflectiveLoader.

![[Pasted image 20250630091526.png]]

When payloads are generated from Cobalt Strike, the relevant information from the listener configuration are patched into the reflective Beacon DLL, which is then converted into position independent code (PIC), or 'shellcode'.  That shellcode is subsequently embedded into each of the payload templates (or 'artifacts') that Cobalt Strike is able to produce - such as the `.exe`, `.dll`, and `.ps1`.  The payloads themselves use process injection techniques to load and execute the shellcode.  In pseudo-code, that may look something like this:

```c
// ReflectiveLoader + Beacon as PIC
unsigned char shellcode[1024] = "<shellcode>";

// Allocate new memory for shellcode
void* ptr = VirtualAlloc(0, 1024, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);

// Copy shellcode into new memory region
memcpy(ptr, shellcode, 1024);

// Create a new thread to execute the ReflectiveLoader
CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)ptr, 0, 0, NULL);
```