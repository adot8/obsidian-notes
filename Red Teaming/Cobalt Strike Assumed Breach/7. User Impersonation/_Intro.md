_User Impersonation_, or _Use Alternate Authentication Material_ as MITRE puts it, is a tactic [T1550](https://attack.mitre.org/techniques/T1550/) for leveraging the credential material obtained during _[Credential Access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b78a7d9f99e969c066ebf)_.  Once an adversary has obtained credentials such as plaintext passwords, hashes, or Kerberos tickets, they need to use that credential to assume the identity of the user to whom they belong.

Before jumping into each technique in detail, it's useful to cover some background information first.

### Logon Sessions

When a user successfully authenticates to a system, the relevant authentication package creates a new logon session for that user.  A logon session holds information such as:

- A unique Logon Identifier (referred to as a LUID).
- The user's username & Security Identifier (SID).
- The authentication package used to authenticate the user.
- The logon time.
- The logon server (i.e. domain controller) used.
- The DNS domain name & User Principal Name (UPN).

When the logon session has been created, the authentication package passes information back to the Local Security Authority (LSA) which it uses to create an access token for the user.

### Access Tokens
An access token contains security information associated with a logon session (i.e. the user).  It holds information including:

- A unique Token ID.
- The ID of the parent logon session (labelled Authentication ID).
- The token type (_primary_ or _impersonation_).
- The user's SID.
- The SIDs of any domain groups the user is a member of.
- A list of privileges (i.e. user right assignments) the user has.
- The token's integrity level (e.g. Low, Medium, High, System).

Every process that starts within the logon session will be assigned this _primary_ access token, and thus every action a process performs will be under the identity of the user.  This token is the "thing" that Windows checks when making access-based decisions when actions are performed against securable objects.  For example, notepad.exe tries to open a text file - Windows reads the information and privileges within the access token and checks it against the permissions of the target file.

Because each process is given its own copy of the access token, a process can modify some security settings in its copy without affecting other processes.  Some applications, such as browsers, do this specifically to make themselves less privileged; whilst others may temporarily enable special privileges to perform sensitive tasks.  Furthermore, a thread within a process can impersonate a different access token, so that it can carry out actions under a different identity, or with different privileges than the main process token.

When you run commands, such as `whoami`, the information returned is generally taken from the primary access token of the process.

### Credential Cache

As we known from the [_OS Credential Dumping_](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=678000345b6876a7e8080887) lesson, LSASS stores credentials (including plaintext, NTLM & AES hashes, and Kerberos tickets) in memory, on behalf of users who have active logon sessions.  When a process attempts to access a network resource, Windows uses the information in its access token to lookup the associated logon session and its associated credential cache.  Those cached credentials are then used to authenticate to the remote service.

![[Pasted image 20250710223058.png]]
