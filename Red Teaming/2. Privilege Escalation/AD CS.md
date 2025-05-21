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

#### Abusing AD CS

Here are various ways of abusing ADCS!  
(See the link to the *"Certified Pre-Owned"* paper in the slide notes):

- Extract user and machine certificates  
- Use certificates to retrieve NTLM hash  
- User and machine level persistence  
- Escalation to Domain Admin and Enterprise Admin  
- Domain persistence  

##### Stealing Certificates
| Method                                   | Description                                                             |
|------------------------------------------|-------------------------------------------------------------------------|
| Exporting with Windows Crypto APIs       | Export certificates with private keys using Windows' crypto APIs        |
| Extracting User Certs via DPAPI          | Extract user certificates with private keys using DPAPI                 |
| Extracting Machine Certs via DPAPI       | Extract machine certificates with private keys using DPAPI              |
| Stealing from Files and Stores           | Steal certificates from files and certificate stores                    |
| Using Kerberos PKINIT                    | Use Kerberos PKINIT to retrieve NTLM hash                               |
##### Persistence
| Method                                   | Description                                                             |
|------------------------------------------|-------------------------------------------------------------------------|
| User Cert Request                        | Maintain user persistence by requesting new certificates                |
| Machine Cert Request                     | Maintain machine persistence by requesting new certificates             |
| Cert Renewal                             | Maintain user or machine persistence by renewing existing certificates  |
