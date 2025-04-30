### Theory

> [!NOTE] **NOTE**
> Trust is a relationship between two domains or forests which allows users of one domain or forest to access resources in the other domain or forest
> - Automatic - Same forest - Parent -> Child
> - Established - External - Forest 
> 
> Trusted Domain Objects (TDOs) represent the trust relationships in a domain

###  Trust Directions

- One-way (Unidirectional)
    - Users in the trusted domain can access resources in the trusting domain but not vice versa
- Two-way (Bidirectional)
    - Users of both domains can access resources in the other domain
- Transitive
    - Can be extended to establish trust relationships with other domains
    - If Domain A is compromised, you can jump to Domain B **AND** Domain C
    - ![[Pasted image 20250430064036.png]]
- Non-transitive
    - Cannot be extended to other domains in the forest. Can be two-way or one-way.
