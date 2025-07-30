Before diving into specific Kerberos abuse primitives, it's useful to take some time to understand how the protocol works under the hood.  We had a brief exposure in the [Credential Access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9) chapter, but there we'll go a little deeper.

Kerberos has replaced NTLM as the default domain authentication protocol since Windows Server 2000.  It works on a system of 'tickets' where principals (users and computers) are authenticated by a central, trusted server.  It permits principals to access services without needing to provide their password over and over again, or for the end-services to know a principal's password.  Kerberos assumes that there is no transport encryption on the network and that packets can be read/modified/replayed.  It therefore relies on shared-secret cryptography to establish a trust relationship with clients.

Before embarking on further explanation, it's useful to introduce some naming conventions.

- #### Key Distribution Center
    
    The KDC is the trusted source for Kerberos authentication and consists of three sub-components:
    
    - A database of all principals and their associated secrets (i.e. password hashes).
        
    - An Authentication Server (AS).
        
    - A Ticket Granting Server (TGS).
        
    
    In a Windows domain, it's the domain controllers that act as the KDC, as every principal and their password hashes are stored in Active Directory.
    
- #### Ticket Granting Ticket
    
    A TGT is provided to a principal after their identity has been verified by the AS component of the KDC.  They are then used in lieu of a principal having to provide their password every time they want to access a service, which is what provides the single sign-on experience in Windows.
    
- #### Service
    
    A service is a resource that can be accessed by a principal.  This can by anything from an SMB share to an MS SQL database.  Each service instance is identified by a unique Service Principal Name (SPN), which are typically of the format `class/instance:port`.
    
- #### Service Ticket
    
    A principal wishing to access a service using Kerberos authentication first needs to request a service ticket from the TGS component of the KDC.  They send their TGT to the KDC as evidence that their identity has been verified.  The Kerberos RFC simply calls these 'service tickets', although they're also known colloquially as 'TGS tickets' or 'ticket granting service tickets'.
    
- #### Privileged Attribute Certificate
    
    The PAC is a structure that can be attached to a ticket which contains additional information about a principal.  When a ticket is issued from a Windows domain controller, it will populate the PAC with information about the principal from Active Directory.  This can include their RID, domain group membership, UAC information, and more.  When presenting a service ticket to a service, the service is able to inspect the PAC to determine the user's privilege without having to query it from AD.  This improves network performance but PAC validation can be enforced if desired.

## Authentication Overview

The following describes the ticket flow from first user login, to accessing a service.

![[Pasted image 20250720223652.png]]

### Authentication Service Exchange

#### 1 & 2. Kerberos Authentication Service Request

The AS exchange with a KDC is initiated by a client when it wishes to obtain a service ticket but the principal's logon session does not currently contain a TGT.  To request a new TGT, the client sends a Kerberos Authentication Service Request (**AS-REQ**) message to the KDC.

The request contains the principal's name, the Kerberos realm (i.e. domain name), the target service's name (always krbtgt for TGTs), and a list of encryption types that the client supports (in order of preference).

On receipt, the KDC will lookup the principal's account in Active Directory.  The default configuration for accounts created in Active Directory is for them to follow a pre-authentication procedure before the KDC will return a valid TGT.  If this is not explicitly disabled for the requesting principal, the KDC responds with a `KDC_ERR_PREAUTH_REQUIRED` error, along with the acceptable pre-authentication mechanisms.

The most commonly used mechanism is an encrypted timestamp.  The client encrypts the current time using a hash derived from the principal's own plaintext password, which is typically AES256 salted with the principal's username and the Kerberos realm name.  Older versions of Windows use RC4 by default instead of AES.

When the KDC receives this new AS-REQ, it will take a copy of the principal's secret (i.e. password hash) from Active Directory and uses it to decrypt the timestamp.  If successful, it assumes that the request is from the genuine user because no other party should know their password.  The KDC then generates a new TGT and logon session key for the principal, and returns them in an **AS-REP** message.

One of the more confusing aspects of Kerberos messages and tickets is that they're partly in plaintext and partly encrypted.  However, different parts are encrypted using different keys, which can be difficult to keep track of.

A TGT can only be used with knowledge of the corresponding logon session key.  One copy of this key is encrypted into the TGT using the KDC's secret (which is the password hash of the krbtgt account in a Windows domain).  And another copy of the key is encrypted using the principal's secret, and is included in the AS-REP.

![[Pasted image 20250720223905.png]]

The client decrypts the logon session key from the **EncASRepPart** of the AS-REP and stores it along with the TGT in the principal's logon session.  The TGT itself cannot be decrypted because the KDC's secret is not known by the client.

>Recall the lesson on [AS-REP Roasting](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9) - it's this EncASRepPart that gets cracked to recover the plaintext password of the user.

#### 3 & 4. Ticket-Granting Service Exchange

Now that the principal has a TGT and logon session key, they are able to request new service tickets.  This happens transparently when the service is accessed (e.g. when requesting to list a file share).   The client constructs a Ticket-Granting Service Request (**TGS-REQ**) message which contains the SPN of the target service (e.g. `cifs/computer.domain.com`), the principal's TGT, and an authenticator.

Because AS-REPs (and thus TGTs) are transmitted over an assumed unencrypted network, Kerberos requires a means to validate that TGS-REQs are coming from the genuine principal, and not a 3rd party that has intercepted a previous AS-REP in transit.  This is the purpose of the authenticator.  The content is not that important for this particular discussion, what is important is that it's encrypted using the principal's current logon session key.  Since this key is never transmitted in the clear, it proves that the TGS-REQ is coming from a legitimate client because no other party should know this key.

Upon receipt of the TGS-REQ, the KDC must first decrypt and inspect the authenticator.  To do that, it needs the logon session key that was used to encrypted it - a copy of which is included inside the EncTicketPart of the TGT.  It therefore uses the krbtgt hash to decrypt the TGT, recovers the session key, and uses it to decrypt the authenticator.  Once the request has been verified, the KDC looks up the requested SPN in Active Directory to determine which principal the service is associated with - this could be a user or computer.

As long as the authenticator and SPN are technically valid, the KDC generates a new service ticket for the requested service, along with a new service session key, and returns them to the client in a **TGS-REP** message.  The KDC also includes a copy of the PAC from the principal's TGT, in the service ticket by default.

>The KDC plays no role in validating what permissions or access (if any) the principal has to the requested service.  That's up to the service to decide in step 5.

As with a TGT, knowledge of the service session key is required to use the service ticket.  The principal's copy is encrypted using their logon session key, and the service's copy is encrypted into the service ticket using the service's secret.  This will be the secret of the user/computer account to which the SPN is associated.

![[Pasted image 20250720224248.png]]

The client can decrypt the service session key from the **EncTGSRepPart** because it has a copy of the logon session key from the AS-REP.  It will store the service ticket and the service session key in the principal's logon session, however it cannot decrypt the service ticket itself because it does not know the service's secret.

> Recall the lesson on [kerberoasting](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9) - it's this EncTicketPart that gets cracked to recover the plaintext password of the service account.

#### 5 & 6. Client/Server Authentication Exchange

The final step in this whole process is to use the service ticket and associated service session key to access the service itself.  If you ever capture this flow in Wireshark, you'll see that there aren't any individual AP-REQ/AP-REP messages on the wire.  This is because they're embedded inside the underlying protocol for the service; facilitated by the Simple and Protected GSSAPI Negotiation Mechanism (SPNEGO) on Windows.  For example, when accessing a file share, the AP-REQ is inside the SMB Session Setup request message.

The **AP-REQ** includes the service ticket and another authenticator, which has been encrypted with the service session key.  The service can decrypt the service ticket because it's encrypted with its own secret.  It then uses the service session key from inside the ticket to decrypt and validate the authenticator.

The service may not need to specifically respond to the AP-REQ.  The AP-REQ has an `ap-options` field, where the client can specify whether mutual authentication is required or not  (this is usually set to true).  In this case, the service must respond to the client with an AP-REP.  It does this by simply re-encrypting the timestamp it received in the AP-REQ authenticator, and sending it back to the client.  The client verifies that it matches the timestamp that it previously sent.  This proves to the the client that it's talking to the legitimate service, because only it would have access to the service session key required to decrypt the authenticator.  The client can terminate further communications with the service if it cannot verify the AP-REQ.

Passing the authenticator checks assures the service that the requesting principal has been authenticated on the domain, but it does not imply authorisation to the resources the service provides.  The service application must make separate decisions based on the principal, the requested operation, and the access control mechanism in use.  In the case of an SMB share, this would be a read/write/modify DACL check on the file share.

When PAC validation is not enforced, the service relies on the information in the service ticket's PAC to know which domain groups, etc, the principal is a member of.  If it is enforced, the service sends an RPC message to the KDC and asks it to verify that the PAC was signed using the realm's secret (the hash of the krbtgt account).  If the PAC signature is not valid (e.g. because the ticket was forged), the KDC returns a message to the service stating that the validation failed, and the service will deny you access.