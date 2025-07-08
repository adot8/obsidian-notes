
OS Credential Dumping is a technique [T1003](https://attack.mitre.org/techniques/T1003/) where an adversary extracts credential material, that are either stored or cached, from the Operating System.  Each sub-technique detailed here requires SYSTEM privileges to the computer.

### LSASS Memory

The Local Security Authority Subsystem Service (LSASS) on Windows is responsible for  verifying the credentials of users when logging in, handling password changes, creating access tokens, and so on.  An adversary may be able to read passwords cached inside memory of the LSASS process [T1003.001](https://attack.mitre.org/techniques/T1003/001/)

Microsoft provides something called the Security Support Provider Interface (SSPI), which is their implementation of the Generic Security Service API (GSSAPI).   SSPs are used to provide different authentication mechanisms for Windows, including:

- _NTLM_ for authentication via NTLM and NTLMv2.
- _Kerberos_ for authentication via Kerberos v5. 
- _Digest_ for Lightweight Directory Access Protocol (LDAP) and web authentication.
- _Schannel_ for authentication via public key cryptography, such as TLS and SSL.
- Credential Security Service Provider (_CredSSP_) for single sign-on with Terminal Services and Remote Desktop sessions.

