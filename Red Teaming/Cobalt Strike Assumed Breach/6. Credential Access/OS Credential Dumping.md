
OS Credential Dumping is a technique [[T1003](https://attack.mitre.org/techniques/T1003/)] where an adversary extracts credential material, that are either stored or cached, from the Operating System.  Each sub-technique detailed here requires SYSTEM privileges to the computer.

### LSASS Memory

The Local Security Authority Subsystem Service (LSASS) on Windows is responsible for  verifying the credentials of users when logging in, handling password changes, creating access tokens, and so on.  An adversary may be able to read passwords cached inside memory of the LSASS process [[T1003.001](https://attack.mitre.org/techniques/T1003/001/)].