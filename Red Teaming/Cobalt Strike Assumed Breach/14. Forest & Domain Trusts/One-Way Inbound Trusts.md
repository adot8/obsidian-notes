
A one-way trust is created when administrators want to share resources with a trusted domain, but they don't want their resources to be accessed from the trusting domain.  You may see trusts setup like this to facilitate one-way migrations or data transfers, etc.

Querying the TDO shows an inbound transitive forest trust with _partner.com_.

```powershell
beacon> getuid
[*] You are CONTOSO\pchilds

beacon> ldapsearch (objectClass=trustedDomain)

name: partner.com
trustDirection: 1
trustAttributes: 8
flatName: PARTNER
```