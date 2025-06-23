### Cyber kill chain

- **Reconnaissance** - scout a target and find potential attack vectors.    
- **Weaponisation** - develop a malicious payload.
- **Delivery** - develop a means of delivering the payload.
- **Exploitation** - the initial attack of delivering the weaponised payload.
- **Installation** - installing persistent malware on the target.
- **Command & Control** - establish a means of controlling compromised targets.
- **Actions on Objectives** - achieve the operational goal (defacement, data theft, etc).

Before threat intelligence as we now know it came about, the kill chain's purpose was to provide a framework for informing defensive measures.  However, by itself it has a few shortcomings.  The most significant being the lack detail - e.g. once an adversary has compromised one or more targets, how do they actually go about achieving their objective?

## Targeted attack lifecycle

Other vendors attempted to come up with their own versions.  One example is Mandiant's 'Targeted Attack Lifecycle', which details the following 8 phases:

- **Initial Reconnaissance** - research the target's systems and employees to develop a methodology for intrusion.
- **Initial Compromise** - execute malicious code on one or more targets via the attack vector planned during phase 1.
- **Establish Foothold** - maintain continued control over a compromised system by installing persistent backdoors.
- **Escalate Privileges** - exploit system vulnerabilities or misconfigurations to obtain local admin access to compromised systems.
- **Internal Reconnaissance** - explore the target's internal infrastructure and environment.
- **Move Laterally** - use credentials obtained from phase 4 to compromise additional systems.
- **Maintain Presence** - maintain highly privileged access to domains and systems.
- **Complete Mission** - accomplish the operational objective.