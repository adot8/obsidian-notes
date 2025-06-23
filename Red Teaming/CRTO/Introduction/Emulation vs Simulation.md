There's a subtle but important difference when it comes to 'emulating' or 'simulating' an adversary.  They have different approaches and achieve different outcomes.

| **Emulation**                                  | **Simulation**                               |
| ---------------------------------------------- | -------------------------------------------- |
| Emulate a specific threat.                     | Simulate a hypothetical threat.              |
| Leverage their known TTPs.                     | Leverage unknown or unique TTPs.             |
| Narrow in scope.                               | Broad in scope.                              |
| Enhance defences against that specific threat. | Enhance defences against a range of threats. |
The purpose of adversary _emulation_ is to develop, test, and tune an organisation's ability to detect and respond to the TTPs of a specific threat.  This provides a focused evaluation of their capabilities against a threat actor that is more likely to target them.  In these scenarios, the red team will leverage threat intelligence (TI) reports to mirror the known TTPs of that actor as closely as reasonably possible.  

Conversely, adversary _simulation_ allows the red team to behave like a completely hypothetical threat, which is less restrictive in terms of the TTPs they can leverage.  This provides an organisation with a broader evaluation of their capabilities and can highlight lesser-known blind spots.

A common strategy for organisations to employ is to leverage adversary emulation to set a baseline capability, and then use adversary simulation to improve that capability against more advanced or a wider variety of TTPs.


> [!NOTE] **Primum non nocere**
> The Latin phrase "primum non nocere" - "first, do no harm"

### Stealth and OPSEC
OPSEC is used to describe the likelihood of actions being observed by enemy intelligence.  From the perspective of a red team, this would be a measure of how likely their actions can be observed and subsequently interrupted by a blue team.

OPSEC only becomes a significant concern during adversary simulation, rather than emulation.  Because when emulating an adversary, the red team is constrained to a specific set of TTPs.  However, during a simulation, the red team's goal is to achieve the operational objective before getting caught by the blue team, so OPSEC becomes a lot more important.

It's therefore crucial that a red teamer understands what indicators they leave behind when carrying out their TTPs.  This course makes a conscious effort to review these indicators.

### Threat intelligence
The consumption and contextualisation of threat intelligence helps an organisation identify emerging threats and their possible mitigations; frame testing (e.g. red team) scenarios; and compare observed activity within their environments with known TTPs and indicators of compromise (IOCs).  IOCs can also be used to tweak intrusion detection systems (IDS), intrusion protection systems (IPS) and firewall rules.