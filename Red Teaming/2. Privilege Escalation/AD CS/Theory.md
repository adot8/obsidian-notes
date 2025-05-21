Active Directory Certificate Services (AD CS) enables use of Public Key Infrastructure (PKI) in active directory forest.AD CS helps in authenticating users and machines, encrypting and signing documents, filesystem, emails and more.

> [!NOTE] **NOTE**
> "AD CS is the Server Role that allows you to build a public keyinfrastructure (PKI) and provide public key cryptography, digital certificates, and digital signature capabilities for your organization."

#### Important shit

| Term                 | Description                                                                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| CA                   | The certification authority that issues certificates. The server with AD CS role (DC or separate) is the CA.                                   |
| Certificate          | Issued to a user or machine and can be used for authentication, encryption, signing, etc.                                                      |
| CSR                  | Certificate Signing Request made by a client to the CA to request a certificate.                                                               |
| Certificate Template | Defines settings for a certificate. Contains information like enrollment permissions, EKUs, expiry, etc.                                       |
| EKU OIDs             | Extended Key Usages Object Identifiers. These dictate the use of a certificate template (Client authentication, Smart Card Logon, SubCA, etc.) |
#### In a nutshell
![[Pasted image 20250520192006.png]]

### Abusing AD CS

Here are various ways of abusing ADCS!  

- Extract user and machine certificates  
- Use certificates to retrieve NTLM hash  
- User and machine level persistence  
- Escalation to Domain Admin and Enterprise Admin  
- Domain persistence  
##### Stealing Certificates (Theft)

| Method | Description                                                      |
| ------ | ---------------------------------------------------------------- |
| THEFT1 | Export certificates with private keys using Windows' crypto APIs |
| THEFT2 | Extract user certificates with private keys using DPAPI          |
| THEFT3 | Extract machine certificates with private keys using DPAPI       |
| THEFT4 | Steal certificates from files and certificate stores             |
| THEFT5 | Use Kerberos PKINIT to retrieve NTLM hash                        |
##### Persistence
| Method   | Description                                                            |
| -------- | ---------------------------------------------------------------------- |
| PERSIST1 | Maintain user persistence by requesting new certificates               |
| PERSIST2 | Maintain machine persistence by requesting new certificates            |
| PERSIST3 | Maintain user or machine persistence by renewing existing certificates |
##### Escalation (ESC)

| Method            | Description                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------ |
| **ESC1  ^**       | Enrollee can request certificate for **any** user                                                                  |
| ESC2              | Certificate template allows any purpose or has **no EKU** (potentially dangerous)                                  |
| **ESC3 ^**        | Request an **Enrollment Agent** certificate and use it to request a cert on behalf of **any** user                 |
| **ESC4 ^**        | **Overly permissive ACLs** on certificate templates                                                                |
| **ESC5 ^**        | Poor access control on **CA server**, **CA computer object**, etc.                                                 |
| ESC6              | **(Patched - May 2022)** Abuse of `EDITF_ATTRIBUTESUBJECTALTNAME2` setting on CA to request certs for **any** user |
| **ESC7 ^**        | Poor access control on **roles** on the CA (e.g., "CA Administrator", "Certificate Manager")                       |
| **ESC8 ^**        | **NTLM relay** to HTTP enrollment endpoints                                                                        |
| ESC9 - similar 10 | No security extension – enrollee can **modify own UPN** to request a cert on behalf of **any** user                |
| ESC10 - similar 9 | **Implicit weak certificate mapping** – enrollee modifies own UPN to request a cert for **any** user               |
| ESC11             | **NTLM relay** to RPC enrollment endpoints                                                                         |
| ESC12             | Steal **CA private key** from Yubico YubiHSM                                                                       |
| ESC13             | Enrollee gets privileges of the **linked group**                                                                   |
| ESC14             | **(To be patched)** Authenticate as the target using a certificate referenced in `altSecurityIdentities` attribute |
| ESC15             | **(Patched - Nov 2024)** EKUwu – Abuse of default version 1 templates to **override EKUs**                         |
##### Domain Persistence
| Method     | Description                                                           |
|------------|-----------------------------------------------------------------------|
| DPERSIST1  | Forge certificates with stolen CA private keys                        |
| DPERSIST2  | Use of malicious root or intermediate CAs                             |
| DPERSIST3  | Backdoor the CA server, its computer object, or related infrastructure |
