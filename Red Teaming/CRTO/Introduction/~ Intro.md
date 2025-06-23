
> [!NOTE] **Note**
> Red Teaming is the process of using tactics, techniques and procedures (TTPs) to emulate a real-world threat, with the goal of measuring the effectiveness of the people, processes and technologies used to defend an environment.

Red teams attempt to breach systems in the same way a real-life attacker would.  This could be through social engineering such as phishing, or physically breaking into a premiss.  They may employ any TTP within their arsenal, or use the TTPs of a specific threat actor.  The latter prepares an organisation for defending itself against threats known to target their industry.  Once a red team has gained access to their target environment, they begin collecting information to help achieve their goal.

The goal of a red team is to achieve an operational objective that has been pre-agreed with the client.  This will vary between clients, but will often entail gaining access to business-critical systems or data.  It should not be generic, such as "get domain admin".  One valuable outcome of a red team assessment is a clear indication of business risk by demonstrating the impact of compromise.  Gaining access to a domain admin user (or the user of a privileged group) may be a stepping stone in achieving the operational objective, but it should not be the objective itself.

## Tactics, techniques and procedures

Tactics, techniques and procedures; often shorted to TTPs; describe the why, the how, and the specific steps an adversary will take to achieve their objective.

- #### Tactics
    A tactic is the overall tactical goal - the reason for performing an action.  An adversary achieves their objective by chaining multiple tactics together such as privilege escalation, credential access, and lateral movement.  The output or result of one tactic often leads into the next (i.e. escalate privileges to dump credentials that are used to move laterally).
    
- #### Techniques
    A technique describes how an adversary will achieve a tactic.  For example, access to credentials could be achieved by extracting them from LSASS memory or by reading them from unsecured locations on disk, such as the Registry.  There are often multiple techniques that could be leveraged to achieve a given tactic.
    
- #### Procedures
    A procedure describes the exact step-by-step process of how a technique is performed.  For example, credentials could be read from LSASS using Mimikatz, or via the Task Manager.  These are two different procedures that achieve the same technique.
![[Pasted image 20250623102432.png]]
## Blue teams

The role of a blue team is to defend an organisation from attack.  Some organisations have dedicated teams such as a Security Operations Centre (SOC); others are made up of personnel from their IT teams; and others may outsource to a Managed Service Provider (MSP).  During a red team assessment, a blue team must detect and respond to the red team incursion via their incident management processes.  At the end of the assessment period, any gaps in detection and response capabilities are reviewed and (hopefully) improvements can be implemented.