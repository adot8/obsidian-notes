The Service for User (S4U) Kerberos extension was introduced in Server 2003 as a replacement for the rather problematic unconstrained delegation.  It provides two new protocols:

- Service for User to Proxy (S4U2proxy).  This allows a service to obtain a service ticket on behalf of a user to a different service (e.g. an HTTP service requesting a service ticket to the MSSQLSvc service).  This is known as "constrained delegation".
    
- Service for User to Self (S4U2self).  This allows a service to obtain a service ticket on behalf of a user to itself.  This is intended to be used when a user authenticates to the service in a way other than Kerberos.  The service can perform an S4U2self to get a service ticket for the user as though they did authenticate using Kerberos, and then use that with S4U2proxy to get a service ticket for another service.  This is known as "protocol transition".
    

The main difference from unconstrained delegation is that S4U attempts to limit the services to which another service can delegate access.  That is to say, instead of being able to request service tickets for any service in the domain, it is limited to the SPNs specified in the configuration.  It also does not rely on the service needing the TGT of a user to work.

Rather than using the TRUSTED_FOR_DELEGATION flag, constrained delegation is configured via the `msDS-AllowedToDelegateTo` attribute of the computer object.  This is a list of SPNs that the computer is allowed to delegate to, e.g. `{ MSSQLSvc/cont-sql-1, ... }`.

These computers can be enumerated using a tool such as `ldapsearch`.

```powershell
beacon> ldapsearch (&(samAccountType=805306369)(msDS-AllowedToDelegateTo=*)) --attributes samAccountName,msDS-AllowedToDelegateTo

sAMAccountName: LON-WS-1$
msDS-AllowedToDelegateTo: cifs/lon-fs-1.contoso.com, cifs/lon-fs-1
```

## Kerberos only (S4U2proxy)

When only Kerberos authentication is enabled (which is the default), the client sends a TGS-REQ for the front-end service and receives a TGS-REP as normal.  The service is then accessed by sending the service ticket in the AP-REQ.  The service will cache the user's service ticket in memory so it can be used later.

When the front-end service needs to access a back-end service on behalf of the user, it sends a TGS-REQ to the KDC asking for a service ticket to the target SPN, and includes a copy of the user's cached service ticket.

The KDC will verify that the msDS-AllowedToDelegateTo attribute of the computer making the request contains the SPN it's requesting a ticket for.  If it is, it returns the service ticket.  The front-end server will then use this service ticket to authenticate to the back-end service as the user.

This is S4U2proxy.

## Protocol transition

Protocol transition is disabled unless the `TRUSTED_TO_AUTH_FOR_DELEGATION` flag on the UserAccountControl attribute of the computer object is explicitly set.  You can grab the current value and compare it to the documented [property flags](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties).  The default UAC value for a computer is 4096 (`WORKSTATION_TRUST_ACCOUNT`).  If this value comes back any different then you know some additional flags are set.
