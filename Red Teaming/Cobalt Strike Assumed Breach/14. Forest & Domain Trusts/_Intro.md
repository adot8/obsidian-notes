
Many Active Directory environments are more complex than a single, isolated, domain.  They have trust relationships with other forests and/or domains.  The purpose of a trust is to allow one forest or domain to share its resources with another.  Trusts can exist between domains in the same forest, between domains in different forests, and even between entire forests.  The following summarises the types of trust you're more likely to encounter:

![[Pasted image 20250724145843.png]]

- #### Parent/Child Trust
    
    A two-way, transitive trust that is automatically created when a new domain is added to an existing tree.
    
- #### Tree-Root Trust
    
    A two-way, transitive trust that is automatically created when a new domain tree is added to an existing forest.
    
- #### External Trust
    
    A one or two-way, non-transitive trust that enables resources to be shared between domains in different forests.
    
- #### Forest Trust
    
    A one or two-way transitive trust that enables resources to be shared between different forests.

### Transitivity

The _transitivity_ of a trust determines whether the trust relationship should extend beyond the two parties with which it was formed.  Imagine a scenario where Domain A has an explicit trust relationship with Domain B; and Domain B has an explicit trust relationship with Domain C.  If these trusts were transitive, then Domain A would also implicitly trust Domain C.

![[Pasted image 20250724151825.png]]

> Two-way trusts do not actually exist in Active Directory, they are just two one-way trusts in opposite directions.

### Trusted Domain Objects

Information about each trust relationship is stored in Active Directory as a Trusted Domain Object (TDO).  This includes the trust type, transitivity, and the shared password used to create it.  The primary domain controller in the trusting domain changes the TDO password every 30 days and propagates it to a domain controller in the trusted domain.  TDO's can be read by querying the `trustedDomain` object class.

```powershell
beacon> ldapsearch (objectClass=trustedDomain)
```

Important attributes include:

- #### cn
    
    This is the the fully qualified domain name (FQDN) of the domain.
    
- #### flatName
    
    This is the NetBIOS name of the domain.
    
- #### trustDirection
    
    Specifies the direction of a trust, where:
    
    - **0** is `TRUST_DIRECTION_DISABLED`.
        
    - **1** is `TRUST_DIRECTION_INBOUND`.
        
    - **2** is `TRUST_DIRECTION_OUTBOUND`.
        
    - **3** is `TRUST_DIRECTION_BIDIRECTIONAL`.
        
    
- #### trustAttributes
    
    These are a set of bitwise flags that define various properties of the trust.  I shan't list them all here, but relevant ones include:
    

- **1** is `TRUST_ATTRIBUTE_NON_TRANSITIVE`.
    
- **4** is `TRUST_ATTRIBUTE_QUARANTINED_DOMAIN` and means SID filtering is in place.
    
- **8** is `TRUST_ATTRIBUTE_FOREST_TRANSITIVE` and means the trust is transitive between two forests.
    
- **32** is `TRUST_ATTRIBUTE_WITHIN_FOREST` and means the trust is between two domains in the same forest.
    
- **64** is `TRUST_ATTRIBUTE_TREAT_AS_EXTERNAL` and means that the trust is between two domains in different forests.  SID filtering is also implied.

### Security Boundaries

In this context, a 'security boundary' defines the scope of authority for administrators in these forests and domains.  It may be obvious to say that an enterprise administrator has authority over their entire forest, and a domain administrator only has authority over their particular domain.  It would therefore be logical to assume that a domain administrator should not be able to administer other domains in their forest, otherwise a security boundary would have been broken.  However, this is not the case.

Officially, a security boundary only exists at the forest level.  It is not possible to prevent administrators from one domain accessing data in another domain, if they are part of the same forest.  This will become particularly evident when we look at how an adversary can hop from a child domain, up to the tree-root.


