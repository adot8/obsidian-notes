![[Pasted image 20250402062331.png]]
Our objective is to `stop an attacker from progressing further up the kill chain`, ideally in one of the earliest stages.

1. **Recon** – Attacker selects a target and gathers intelligence through passive (LinkedIn, job ads, company websites) or active (scanning web apps, probing IPs) methods.
    
2. **Weaponize** – Crafting lightweight, undetectable malware tailored to the target’s defenses (AV/EDR) to establish remote access and persistence.
    
3. **Delivery** – Payload is delivered via phishing, malicious websites, social engineering calls, or physical media (e.g., USB).
    
4. **Exploitation** – Payload executes to gain access or control over the target system.
    
5. **Installation** – Malware establishes persistence using droppers, backdoors, or rootkits to evade detection and maintain access.
    
6. **Command & Control** – Attacker remotely controls the compromised system, deploying additional tools to maintain access.
    
7. **Action (Objective)** – The attacker achieves their goal: data exfiltration, privilege escalation, ransomware deployment, etc.

It is important to understand that adversaries won't operate in a linear manner (like the cyber kill chain shows). Some previous cyber kill chain stages will be repeated over and over again. If we take, for example, the `installation` stage of a successful compromise, the logical next step for an adversary going forward is to initiate the `recon` stage again to identify additional targets and find vulnerabilities to exploit, so that he moves deeper into the network and eventually achieves the attack's objective(s).