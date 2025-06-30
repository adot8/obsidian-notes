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
