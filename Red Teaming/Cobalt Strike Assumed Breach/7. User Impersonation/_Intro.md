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