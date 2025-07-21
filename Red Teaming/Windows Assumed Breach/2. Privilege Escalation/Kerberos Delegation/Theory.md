Kerberos Delegation allows to "reuse the end-user credentials to access
resources hosted on a different server".

Lets say there is a user that wants accesses the `Second Hop database` server through the `First Hop web server`. How will the `Second Hop database server` know which user is accessing it for authorisation? This can be done by the `First Hop web sever` impersonating/re-using the end-users credentials to access the `Second Hop database server`... EASY!

User impersonation is the goal of delegation
![[Pasted image 20250514195150.png]]

There are two types of Kerberos Delegation:
- General/Basic or `Unconstrained Delegation` - Allows the first hop(web server in our example) to request access to any service on any computer in the domain
- `Constrained Delegation` - Allows the first hop to request access only to specified services on specified computers. If Kerberos authentication is not used to authenticate to the first hop, Protocol Transition is used to transition the request to Kerberos

---
### Unconstrained Delegation
Allows delegation to any service to any resource on the domain as a
user.

When unconstrained delegation is enabled, the DC places user's TGT inside TGS. On the first hop, the TGT is extracted from TGS and stored in LSASS. This way the server can reuse the user's TGT to access any other resource as the user

![[Pasted image 20250514201409.png]]

### Unconstrained Delegation Coercion
Certain Microsoft services and protocols allow any authenticated user to force a machine to connect to a second machine

As of January 2025, following protocols and services can be used for coercion

| Protocol                     | Service        | Default on Server IS      | Ports Required |
| ---------------------------- | -------------- | ------------------------- | -------------- |
| MS-RPRN                      | Print Spooler  | Yes                       | 445 (SMB)      |
| MS-WSP                       | Windows Search | No (Default on Client OS) | 445 (SMB)      |
| MS-DFSNM (MDI detects this)) | Windows Search | No                        | 445 (SMB)      |
We can force the dcorp-dcto connect to dcorp-appsrvby abusing the Printer bug (MS-RPRN) or if enabled, other services

![[Pasted image 20250514202802.png]]

---
### Constrained Delegation
Allows access only to specified services on specified computers as a
user.

Protocol Transition is used when a user authenticates to a web service without using Kerberos (like a regular login page) and the web service makes requests to the a database server to fetch results based on the user's authorization.

- To impersonate the user, Service for User (`S4U`) extension is used which provides two extensions:
	- Service for User to Self (`S4U2self`) - Allows a service to obtain a forwardable TGS to itself on behalf of a user with just the user principal name **without supplying a password**
	- Service for User to Proxy (`S4U2proxy`) - Allows a service to obtain a TGS to a second service on behalf of a user. Which second service? This is controlled by `msDS-AllowedToDelegateTo` attribute. This attribute contains a list of SPNs to which the user tokens can be forwarded.


> [!NOTE] **NOTE**
> Essentially goes like this:
> **Joe**: *Logs into web server with regular login page (Not kerberos compatible)*
> 
> **Web server (websvc)**: Hey DC, I got this user signed in can I get a TGT for Joe? I only got his username lol
> 
> **KDC**: I see you got the `TRUSTED_TO_AUTHENTICATE_FOR_DELEGATION` attribute type shit and Joe not blocked for delegation or nothin... Heres a forwardable ticket (`S4U2Self`) for Joes account type shit
> 
> **Web server (websvc):** Aight bet bet, lemme throw this forwardable TGT back at you and get a mf TGS for `CIFS/dcorp-mssql.dollarcorp.moneycorp.local`
> 
> **KDC:** Shiiiii... lemme check if dat service (`CIFS/dcorp-mssql.dollarcorp.moneycorp.local`) be in yo `msDS-AllowedToDelegateTo` list websvc.... YOU GOOD KING YOU GOOD
> **KDC**: *Returns a TGS to the webserver for `dcorp-mssql`*
> 
> **Web server (websvc):** *Authenticates to `dcorp-mssql` as Joe using the supplied TGS *
> 


![[Pasted image 20250515062107.png]]

To abuse constrained delegation in above scenario, we need to have access to the websvc account. If we have access to that account, it is possible to access the services listed in `msDS-AllowedToDelegateTo` of the websvc account as **ANY user**

**THE SPN VALUE IN THE TGS IS CLEAR-TEXT WHICH MEANS THAT WE CAN CHANGE THE ALLOWED `CIFS/dcorp-mssql.dollarcorp.moneycorp.local` TO `HTTP/dcorp-mssql.dollarcorp.moneycorp.local` IF WE WANTED TO** 

---
### Resource-based Constrained Delegation
This moves delegation authority **from the Domain Admins** to the **resource/service administrator**.

Instead of SPNs on `msDs-AllowedToDelegatTo` on the **FIRST HOP**  front-end service like web service, access in this case is controlled by security descriptor of `msDS-AllowedToActOnBehalfOfOtherIdentity` (visible as `PrincipalsAllowedToDelegateToAccount`) on the **SECOND HOP** - resource/service like SQL Server service.

That is, the resource/service administrator can configure this delegation
whereas for other types, SeEnableDelegation privileges are required
which are, by default, available only to Domain Admins.

**Abusing RBCD is the same where you can impersonate any user within the domain**

---
To abuse RBCD in the most effective form, we just need two privileges.
- Write permissions over the target service or object to `configuremsDS-AllowedToActOnBehalfOfOtherIdentity` (**configure RBCD on the host**)
- Control over an object which has SPN configured (like admin access to a domain joined machine or ability to join a machine to `domain-ms-DS-MachineAccountQuota` is 10 for all domain user

> [!NOTE] **NOTE**
> You're essentially first configuring RBCD and then abusing it after
