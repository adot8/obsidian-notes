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
