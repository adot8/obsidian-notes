A diamond ticket is created by decrypting a valid TGT, making changes to
it and re-encrypt it using the AES keys of the krbtgt account.

Once again, the persistence lifetime depends on krbtgt account.
A diamond ticket is more **OPSEC** safe as it has a valid ticket times because a TGT issued by the DC is modified

Golden ticket was a TGT forging attacks whereas diamond ticket is a TGT
modification attack


![[Pasted image 20250510210937.png]]