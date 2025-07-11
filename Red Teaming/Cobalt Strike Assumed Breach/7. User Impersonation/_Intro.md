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