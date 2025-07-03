Beacon itself is a Windows DLL.  If you picture the 'modules' (or DLLs) loaded into a Windows process, you expect to see candidates such as ntdll.dll, kernel32.dll and more.  These DLLs are loaded by the process to access required functionality within them.  To execute a Beacon payload, we need to have its DLL loaded into a process and (usually) kick off a new thread to call its entry point.

![[Pasted image 20250626155513.png]]

Windows DLLs are usually loaded from disk, typically from `%windir%\system32`, but this is not ideal as we don't want to rely on dropping a Beacon DLL to disk before it can be executed.  Instead, Beacon is implemented as a reflective DLL, based on [Stephen Fewer's work](https://github.com/stephenfewer/ReflectiveDLLInjection).  This is a type of DLL that can be loaded into a process purely from memory, rather than from disk.  This requires the DLL to have a reflective loader, which is responsible for re-loading its own image into memory and performing tasks such as loading dependant modules.  The 'overview' section of Stephen's repository goes through the various steps in more detail.

The following illustrates an over-simplified PE structure.

![[Pasted image 20250626155754.png]]

Beacon's DLL exports a function called **ReflectiveLoader**, which when called, walks back over its own image, maps a new copy of itself into memory, and calls its entry point.  Because the DLL is exported as shellcode, another small shellcode stub is written over the DOS Header.  This shellcode is responsible for jumping code execution to the exported ReflectiveLoader.

![[Pasted image 20250630091526.png]]

> **Note:** An explanation can also be found here https://www.youtube.com/watch?v=p-ufU9W1i7Q

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

### The 'new' prepended loader

Since Cobalt Strike 4.11, Beacon has begun to use a new style of reflective loader based on the one used by DoublePulsar.  This was built by the NSA and leaked by The Shadow Brokers hacker group.  The loader is famous for its association with EternalBlue and WannaCry.

This is a 'prepended' loader, which means the reflective loader is prepended to the front of a PE rather than being part of it.  The following diagram from Fortra shows the difference between the two loader styles.

![[Pasted image 20250630093608.png]]

A prepended loader is overall more stealthy and flexible than a stomped loader.  It does not require a shellcode stub to be written over the PE's DOS headers; it can reflectively load any PE; it doesn't require the PE to export any functions necessary for the reflective loading process; and it gives more options to obfuscate the PE.  The PE itself can be encoded, encrypted, compressed, etc, and it will still work as long as the reflective loader knows how to undo those obfuscations.

### Staged vs stageless payloads

It's common for attack frameworks, such as Metasploit and Cobalt Strike to decouple 'exploits' from 'payloads'. Imagine an application that has a code execution vulnerability due to a buffer overflow - the 'exploit' is the code that triggers the overflow and the 'payload' is the code you want executed as a result (e.g. Meterpreter or Beacon shellcode). The size of payload that you can deliver may be limited by factors such as the nature of the vulnerability itself, or the channel it's accessible on. If you're old like me, you may remember MS08-067 - an infamous remote code execution (RCE) vulnerability in Windows. This was a stack corruption vulnerability which had to be exploited within a limit of ~400 bytes.

To combat these situations, 'payload staging' became popular. This consists of a small program (the stager) that when executed, reaches out to fetch the full payload (the stage) over a different channel from the exploit, such as HTTP, inject it into a new region of memory and pass execution to it. Payload staging is good for getting around these types of exploit restrictions, but they also have several drawbacks.

#### Payload security

When first started, a Cobalt Strike team server will generate a new unique public/private keypair, and every stageless payload generated by that server will have the server's public key embedded into it. Beacon uses this key to encrypt its metadata when talking to the team server, ensuring that it can only communicate with the team server that it was generated from. In addition, every Beacon uses a unique session key that is used to encrypt/decrypt the tasking/output data exchanged with the team server. That session key itself is transmitted inside the encrypted metadata, so that only the correct team server can encrypt and decrypt C2 traffic for a Beacon. The upshot of this is that you cannot easily man-in-the-middle a stageless payload to hijack control of it. However, due to their small size, stagers do not carry the same security. When executed, the stager will reach out to the host/port that it was configured with to receive the rest of the payload stage. There is no validation by the stager to ensure that it's talking to a legitimate team server, which does make them suspectable to hijacking when first executed.

#### OPSEC

To keep the stagers as small as possible, they must omit steps that we may consider these days as 'good practice'. For instance, the full stageless payloads avoid the use of RWX (read, write, execute) memory because it stands out as anomalous in most Windows processes. Instead, they allocate memory as RW first, and then flips it to RX prior to execution. Payload stagers don't do this because it requires an additional API call, which requires more code. Stagers are often built in hand-optimised assembly, so every byte that can be saved, is.

To give an idea on size difference - today, a Beacon stager is ~890 bytes and a full Beacon stage is ~307200 bytes.  That's ~345 times larger.

### Generating payloads

Beacon payloads can be generated from the Payloads menu.

#### HTML application
This option generates a payload in `.hta` format, which is a combination of HTML and VBScript.  This is a scripting language support by Internet Explorer, so became a popular payload format for phishing attacks.  The dialog prompts you to select a listener and the method of execution.

![[Pasted image 20250630094324.png]]

The options are:

- ##### Executable
    Drop an exe stager to disk and run it.
    
- ##### PowerShell
    Uses powershell.exe to run a stager in memory.
    
- ##### VBA
    Uses a VBA macro to run a stager in memory.  The macro is executed by using VBScript to instantiate an instance of _Excel.Application_ and creating a workbook instance. Requires Microsoft Office to be installed on the target.
    
This HTA always delivers an x86 Beacon payload.

#### MS Office macro
This option will produce a VBA macro that you can paste into an Office document - another popular phishing payload format.

![[Pasted image 20250630094522.png]]

This macro always delivers an x86 Beacon payload.

#### Stager payload generator
These payload generators produce source code files for a variety of languages, equivalent to Metasploit's `msfvenom` utility.  These are useful when you want a byte array that you can drop into your own shellcode runners.

![[Pasted image 20250630094609.png]]

For example, outputting a stager in C produces the following:
```c
/* length: 888 bytes */
unsigned char buf[] = "\xfc\x48\x83\xe4[...snip...]\x12\x1d\x9b\x09";
```

> Stager payloads do not allow you to specify any additional configuration details, such as guardrails.

#### Stageless payload generator
The stageless payload generator is practically identical to the stager payload generator, but offers more options.
![[Pasted image 20250630094725.png]]
##### Exit Function

This setting controls the API that gets calls when Beacon's `exit` command is executed.  **Process** calls [ExitProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-exitprocess), which will terminate the whole process that Beacon is running in; whilst **Thread** calls [ExitThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-exitthread), which only terminate the thread running Beacon.  ExitProcess (xprocess) and ExitThread (xthread) payloads both have their uses. 
> If Beacon is injected into a process that is already running on a target system, use xthread.  If injected into a process executed by yourself, then use xprocess.

##### System Call
This tells Beacon to use system calls instead of regular Win32 APIs for its internal operation.  The two options are:

- **Direct** uses the Nt* version of the function.
- **Indirect** jumps to the appropriate instruction within the Nt* version of the function.

#### Windows stager/stageless payload
These options are similar to the stager/stageless payload generators, but instead of outputting raw source files, they provide pre-built executables.  The output options are:

- ##### Windows EXE
    
    A standard Windows executable.
    
- ##### Windows Service EXE
    
    A Windows executable specifically designed to interact with the Service Control Manager.
    
- ##### Windows DLL
    
    A DLL that contains several exports that can be called via utilities such as `rundll32` and `regsvr32`.
    

These payloads can also be signed using a code-signing certificate.

### Windows Stageless Generate All Payloads
This option will generate all possible stageless payload variants for every available listener in both x86 and x64 architectures.
![[Pasted image 20250630095040.png]]