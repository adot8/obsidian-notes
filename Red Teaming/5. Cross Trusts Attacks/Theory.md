`sIDHistory` is a user attribute designed for scenarios where a user is moved from one domain to another. When a user's domain is changed, they get a new SID and the old SID is added to `sIDHistory`

`sIDHistory` can be abused in two ways of escalating privileges within a forest:

1. krbtgt hash of the child
2. Trust tickets

**No Domain Controllers can issue service tickets outside of it's own realm, this is why inter-realm TGTs need to be issued for services in another domain **

**A `TRUST KEY` is used to encrypt the tickets for cross domain TGTs - Only the Domain Controllers have access to this**

![[Pasted image 20250517105517.png]]

---
### Abuse

1. Obtain **Trust Key** (rc4 is used by default)
2. Forge inter-realm TGT to say that the users `sIDHistory` is set to `509` (Enterprise Admins group) - This gives us Enterprise Admin privileges
3. The parent/trusted DC only validates if it's encrypted with the **Trust Key** and not the contents

![[Pasted image 20250517111125.png]]