# Service Misconfigurations
### Authentication
#### Weak/Default credentials
As administrators, we need to define password policies that apply to software tested or installed in our environment
```shell
admin:admin
admin:password
admin:<blank>
root:12345678
administrator:Password
```
#### Anonymous Authentication
Another misconfiguration that can exist in common services is anonymous authentication. The service can be configured to allow anonymous authentication, allowing anyone with network connectivity to the service without being prompted for authentication.
#### Misconfigured Access Rights
Let's imagine we retrieved credentials for a user whose role is to upload files to the FTP server but was given the right to read every FTP document. The possibility is endless, depending on what is within the FTP Server. We may find files with configuration information for other services, plain text credentials, usernames, proprietary information, and Personally identifiable information (PII).

Administrators need to plan their access rights strategy, and there are some alternatives such as [Role-based access control (RBAC)](https://en.wikipedia.org/wiki/Role-based_access_control), [Access control lists (ACL)](https://en.wikipedia.org/wiki/Access-control_list). If we want more detailed pros and cons of each method, we can read [Choosing the best access control strategy](https://authress.io/knowledge-base/role-based-access-control-rbac) by Warren Parad from Authress.

### Unnecessary Defaults
The initial configuration of devices and software may include but is not limited to settings, features, files, and credentials. Those default values are usually aimed at usability rather than security. Leaving it default is not a good security practice for a production environment. Unnecessary defaults are those settings we need to change to secure a system by reducing its attack surface.

[Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/) are part of the [OWASP Top 10 list](https://owasp.org/Top10/). Let's take a look at those related to default values:

- Unnecessary features are enabled or installed (e.g., unnecessary ports, services, pages, accounts, or privileges).
- Default accounts and their passwords are still enabled and unchanged.
- Error handling reveals stack traces or other overly informative error messages to users.
- For upgraded systems, the latest security features are disabled or not configured securely.

### Preventing Misconfigurations
Once we have figured out our environment, the most straightforward strategy to control risk is to lock down the most critical infrastructure and only allow desired behavior. Any communication that is not required by the program should be disabled. This may include things like:

- Admin interfaces should be disabled.
- Debugging is turned off.
- Disable the use of default usernames and passwords.
- Set up the server to prevent unauthorized access, directory listing, and other issues.
- Run scans and audits regularly to help discover future misconfigurations or missing fixes.

The OWASP Top 10 provides a section on how to secure the installation processes:

- A repeatable hardening process makes it fast and easy to deploy another environment that is appropriately locked down. Development, QA, and production environments should all be configured identically, with different credentials used in each environment. In addition, this process should be automated to minimize the effort required to set up a new secure environment.
- A minimal platform without unnecessary features, components, documentation, and samples. Remove or do not install unused features and frameworks.
- A task to review and update the configurations appropriate to all security notes, updates, and patches as part of the patch management process (see A06:2021-Vulnerable and Outdated Components). Review cloud storage permissions (e.g., S3 bucket permissions).
- A segmented application architecture provides effective and secure separation between components or tenants, with segmentation, containerization, or cloud security groups (ACLs).
- Sending security directives to clients, e.g., security headers.
- An automated process to verify the effectiveness of the configurations and settings in all environments.