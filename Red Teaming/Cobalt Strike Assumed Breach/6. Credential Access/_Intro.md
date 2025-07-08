_Credential Access_ is a tactic [TA0006](https://attack.mitre.org/tactics/TA0006/) where an adversary attempts to steal the authentication material of users.  This can include usernames, plaintext passwords, password hashes, and Kerberos tickets.  These are often used for other tactics, such as [lateral movement](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b793699b3a08430017d88), as leveraging legitimate credentials to access resources is less suspicious, and often easier to perform, than exploiting software vulnerabilities.

Some credential access techniques can be performed as a standard user; others require local administrative privileges; and others require Domain Admin privileges (I've elected to cover these in the [Domain Dominance](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=6731fd8f542c8a0baa0af1fa) chapter).

This chapter will cover the following techniques:

- Credentials from Web Browsers [T1555.003](https://attack.mitre.org/techniques/T1555/003/)
- Windows Credential Manager [T1555.004](https://attack.mitre.org/techniques/T1555/004/)
- LSASS Memory [T1003.001](https://attack.mitre.org/techniques/T1003/001/)
- Security Account Manager [T1003.002](https://attack.mitre.org/techniques/T1003/002/)
- LSA Secrets [T1003.004](https://attack.mitre.org/techniques/T1003/004/)
- Cached Domain Credentials [T1003.005](https://attack.mitre.org/techniques/T1003/005/)
- Kerberoasting [T1558.003](https://attack.mitre.org/techniques/T1558/003/)
- AS-REP Roasting [T1558.004](https://attack.mitre.org/techniques/T1558/004/)