
Before diving into the individual types of trust, it's useful to understand how Kerberos works across different realms.  If you need a refresher on how Kerberos works in the same realm, go back to the [Kerberos introduction.](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b79df7385fdbe1c0e7737)  Essentially, a principal obtains a TGT from the KDC in their realm and uses it to request service tickets for services in that same realm.  However, the exact same process is not possible when requesting service tickets from a different realm.  Typical TGTs issued in a trusted realm cannot be decrypted by a trusting realm because it doesn't have access to the same krbtgt secret.  For this reason, an inter-realm key is required to bridge this cryptographic gap.

> Fun fact - all parent/child trusts in a forest use the same inter-realm key, which is actually how trust transitivity is implemented.  Transitive trusts share the same inter-realm key, whereas unique keys are created for each new non-transitive trust.

When a client wishes to access a service in a trusting realm, it will send a TGS-REQ to its own KDC first.  This request includes a copy of their normal TGT from the trusted realm, the `realm` field is also set to that of the trusted realm, and the `sname` field contains the SPN of the target service.

![[Pasted image 20250724160115.png]]

Upon receipt, the KDC will spot that the requested service is in a different realm; so instead of returning a TGS-REP containing a service ticket for the target service (which it can't do because it doesn't know the foreign SPNs secret); it returns a TGS-REP containing an inter-realm TGT.  This is simply a TGT that has been issued by the trusted realm, but is encrypted using the shared inter-realm key instead of the krbtgt secret.

> Microsoft describes this as a 'TGS referral', so you may find resources that describe an inter-realm ticket as a 'referral' ticket.

The TGT that comes back has its realm field set to the trusted realm, but the SPN is set to the krbtgt service of the trusting realm.

![[Pasted image 20250724160354.png]]

The client then uses this inter-realm TGT to send another TGS-REQ, this time directly to the KDC of the trusting realm.

![[Pasted image 20250724160434.png]]

The foreign KDC will decrypt the inter-realm TGT using its copy of the inter-realm key and returns the service ticket in a TGS-REP.  A high-level summary of this flow is as follows:

![[Pasted image 20250724160507.png]]