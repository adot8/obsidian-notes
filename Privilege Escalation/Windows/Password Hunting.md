#### Lazagne
Dump all cached credentials using [Lazagne](https://github.com/AlessandroZ/LaZagne)
```powershell
lazagne.exe all
```
#### Findstr
```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```
Key terms to search for

| Passwords     | Passphrases  | Keys        |
| ------------- | ------------ | ----------- |
| Username      | User account | Creds       |
| Users         | Passkeys     | Passphrases |
| configuration | dbcredential | dbpassword  |
| pwd           | Login        | Credentials |
#### Windows Search 
![[Pasted image 20250114061203.png]]