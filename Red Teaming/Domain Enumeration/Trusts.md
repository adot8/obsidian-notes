### PowerView
```powershell
Get-DomainTrust
Get-DomainTrust -Domain us.dollarcorp.moneycorp.local
```

Forest mapping

Details about current Forest **AND** external Forests
```powershell
Get-Forest
Get-Forest -Forest eurocorp.local
```


> [!NOTE] FILTER_SIDS = External Trusts
> ```powershell
> Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} ?{$_.TrustAttributes -eq "FILTER_SIDS"}
> ```

All domains in current Forest
```powershell
Get-ForestDomain
Get-ForestDomain -Forest eurocorp.local
```

All global catalogs for the current forest
```powerhshell
Get-ForestGlobalCatalog
Get-ForestGlobalCatalog -Forest eurocorp.local
```

Enumerate trusts for a trusting / external forest
```powershell
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

Map trusts of a forest
```powershell
Get-ForestTrust
Get-ForestTrust -Forest eurocorp.local
```

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

### Types of Trusts
- Default/Automatic Trusts
    - Parent -> Child Trusts
        - **adot8.com** can access resources from **pwned.adot8.com**
- Tree-root trust (Tree -> Tree)
    - It is created automatically whenever a new domain tree is added to a forest root.
    - **ALWAYS** two way transitive
    - Compromise Z, you can hop to Y then A then B and C

> [!NOTE] **IMPORTANT**
> **YOU ONLY NEED ONE DOMAIN ADMIN TO COMPROMISE THE WHOLE FOREST**
> **IF THE ENTERPRISES FOREST IS HUGE, SPLIT IT INTO MULTIPLE FORESTS**

![[Pasted image 20250430064251.png]]
- External Trusts (Forest root -> External Child)
    - Between two domains in different forests when forests do not have a trust relationship
    - Can be one-way or two-way and is non-transitive

Non transitive makes it so if you compromise Z you can compromise Y **BUT** you can only access **EXPLICITLY** allowed resources in B and C
![[Pasted image 20250430064318.png]]

- Forest Trusts
    - Between forest root domains
    - Cannot be extended to a third forest (no implicit trust)
    - One-way or two-way transitive... Need explicit permission for resources
![[Pasted image 20250430064337.png]]