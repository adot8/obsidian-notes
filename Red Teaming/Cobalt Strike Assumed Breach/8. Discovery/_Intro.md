_Discovery_ is a tactic [TA0007](https://attack.mitre.org/tactics/TA0007/) where an adversary attempts to enumerate information about the environment they're operating in.  For instance, after [Credential Access](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b78a7d9f99e969c066ebf) and [User Impersonation](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=6731fbb09d5d68a3780e8427), an adversary needs to know what systems and resources (if any) those users have access to.  They may also be able to discover additional misconfigurations and vulnerabilities in the network, which they may be able to exploit to achieve their goals.  These are usually prerequisite steps to [Lateral Movement](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=674b793699b3a08430017d88).

Basic queries against Active Directory, such as those for users, groups, OUs, GPOs, etc, can be performed as a standard domain user.  The adversary wants to know things like which domain groups users are a member of and which, if any, computers on the network they have local admin access to.

Most domain enumeration tools leverage LDAP (Lightweight Directory Access Protocol) or LDAPS (LDAP over SSL) to query Active Directory.  Another protocol that has gained popularity is ADWS (Active Directory Web Services), which wraps LDAP queries in HTTP requests.

### BloodHound
Those with a pentesting background often ask why we don't just run a BloodHound collector like [SharpHound](https://github.com/SpecterOps/SharpHound) or [BloodHound.py](https://github.com/dirkjanm/BloodHound.py).  Where this may be appropriate when simulating an adversary that is known to do so, it's often too risky when emulating a more advanced threat.  Many security vendors have signatured the hardcoded queries built into the data collectors; the network traffic for session enumeration; and may alert large, slow, and inefficient data retrievals from Active Directory.  LDAP queries can be logged on the client-side through sources such as the LDAP Client ETW provider; and on the server-side through [performance](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/ldap-queries-run-slowly) events.

BloodHound's graphing capabilities are undoubtably invaluable for identifying attack paths, but running the default collectors will almost certainly get you caught by a mature security team.  However, this is not to say that you cannot use BloodHound at all.  The 'solution' is to perform the data collection in a way that doesn't trigger these detection strategies - that could be through different LDAP queries; slower, more targeted data collection; or even using a complete different protocol.

A popular approach is to run your own data collection queries using a tool like [ldapsearch](https://github.com/trustedsec/CS-Situational-Awareness-BOF/tree/master) or [pyldapsearch](https://github.com/Tw1sm/pyldapsearch), and then use [BOFHound](https://github.com/coffeegist/bofhound) to parse the output into BloodHound-compatible JSON files.  These can then then be loaded into the UI which will populate the cypher database, allowing BloodHound to query them as normal.

BloodHound CE runs in Docker.

![[Pasted image 20250713221846.png]]

![[Pasted image 20250713221850.png]]