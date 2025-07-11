
_Token Impersonation_ is a technique where an adversary impersonates an access token belonging to another user.  Those tokens can be created if the adversary knows the plaintext credentials of a user, or they can be stolen from processes running in other logon sessions.

### Make Token
This is a technique where an adversary uses the plaintext credentials of a user to create a new access token, and then impersonates it.  This is typically done using the [LogonUserA](https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-logonusera) and [ImpersonateLoggedOnUser](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-impersonateloggedonuser) APIs.  This capability is built into Beacon via the `make_token` command.

```powershell
beacon> make_token CONTOSO\rsteel Passw0rd!
[+] Impersonated CONTOSO\rsteel (netonly)
```

> This technique **_does not_** require a high-integrity context.

Cobalt Strike updates the 'user' field to show that it's currently impersonating a different context.

![[Pasted image 20250710223348.png]]

### Steal Token
This is a technique where an adversary "steals" the primary access token from a process running as a different user.  The steps to carry this out are:

- [OpenProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocess) to obtain a handle to the target process.
- [OpenProcessToken](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-openprocesstoken) to obtain a handle to its primary access token.
- [DuplicateToken](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-duplicatetoken) to duplicate the primary token into an impersonation token.
- [ImpersonateLoggedOnUser](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-impersonateloggedonuser) to use the impersonation token.

This capability is built into Beacon via the `steal_token` command.  Identify a target process using the `ps` command or Process Explorer first.

```powershell
beacon> ps

 PID   PPID  Name                       Arch  Session     User
 ---   ----  ----                       ----  -------     ----
 5248  1864  cmd.exe                    x64   0           CONTOSO\rsteel
 5256  5248      conhost.exe            x64   0           CONTOSO\rsteel
 5352  5248      mmc.exe                x64   0           CONTOSO\rsteel
 
beacon> steal_token 5248
[+] Impersonated CONTOSO\rsteel
```

> This technique **_does_** require a high-integrity session.

### RevertToSelf
To instruct Beacon to stop impersonating a token, whether made or stolen, run the `rev2self` command.  This calls the [RevertToSelf](https://learn.microsoft.com/en-us/windows/win32/api/securitybaseapi/nf-securitybaseapi-reverttoself) API under the hood.

```powershell
beacon> rev2self
```

### Token Store

The downside of the steal_token command is that if you drop impersonation and the process subsequently closes, you can no longer impersonate it again.  Beacon has a token "store" where you can permanently hold a reference to the token, even after the process has closed.  This works because tokens are reference counted by the kernel, so they won't be disposed if something is holding an active handle to it.

The `token-store` of command is used to manage tokens, such as:

- `token-store steal` to steal a token and add it to the store.
    
- `token-store show` to print the tokens in the store.
    
- `token-store use` to use a token in the store.
    
- `token-store remove` to remove a token from the store.