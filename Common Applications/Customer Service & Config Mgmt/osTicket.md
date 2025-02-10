### Footprinting + Enumeration
`osTicket `sets a cookie named `OSTESSID` during when visited
![[Pasted image 20250210055808.png]]
Most `osTicket` installs will showcase the `osTicke`t logo with the phrase `powered by` in front of it in the page's footer. The footer may also contain the words `Support Ticket System`.

`osTicket` is a web application that is highly maintained and serviced. If we look at the [CVEs](https://www.cvedetails.com/vendor/2292/Osticket.html) found over decades, we will not find many vulnerabilities and exploits that osTicket could have.

### Exploitation

#### Obtaining a Valid Company Email
If we come across a customer support portal during our assessment and can submit a new ticket, we may be able to obtain a valid company email address.

![](https://academy.hackthebox.com/storage/modules/113/new_ticket.png)

This is a modified version of osTicket as an example, but we can see that an email address was provided.

![](https://academy.hackthebox.com/storage/modules/113/ticket_email.png)

Now, if we log in, we can see information about the ticket and ways to post a reply. If the company set up their helpdesk software to correlate ticket numbers with emails, then any email sent to the email we received when registering, `940288@inlanefreight.local`, would show up here. With this setup, if we can find an external portal such as a Wiki, chat service (Slack, Mattermost, Rocket.chat), or a Git repository such as GitLab or Bitbucket, we may be able to use this email to register an account and the help desk support portal to receive a sign-up confirmation email.

![](https://academy.hackthebox.com/storage/modules/113/ost_tickets.png)

#### Sensitive Data Exposure
Let's say we are on an external penetration test. During our OSINT and information gathering, we discover several user credentials using the tool [Dehashed](http://dehashed.com/)
```bash
adot8@htb[/htb]$ python3 dehashed.py -q inlanefreight.local -p

id : 5996447501
email : julie.clayton@inlanefreight.local
username : jclayton
password : JulieC8765!
hashed_password : 
name : Julie Clayton
vin : 
address : 
phone : 
database_name : ModBSolutions
```
This dump shows cleartext passwords for two different users: `jclayton` and `kgrimes`. At this point, we have also performed subdomain enumeration and come across several interesting ones
```shell
adot8@htb[/htb]~ cat ilfreight_subdomains

vpn.inlanefreight.local
support.inlanefreight.local
```

`Support.inlanefreight.local` is hosting an osTicket instance, and `vpn.inlanefreight.local` is a Barracuda SSL VPN web portal that does not appear to be using multi-factor authentication.
![[Pasted image 20250210060221.png]]

Let's try the credentials for `jclayton`. No luck. We then try the credentials for `kgrimes` and have no success but noticing that the login page also accepts an email address, we try `kevin@inlanefreight.local` and get a successful login!

The user `kevin` appears to be a support agent but does not have any open tickets. Perhaps they are no longer active? In a busy enterprise, we would expect to see some open tickets. Digging around a bit, we find one closed ticket, a conversation between a remote employee and the support agent.

![](https://academy.hackthebox.com/storage/modules/113/osticket_ticket.png)

The employee states that they were locked out of their VPN account and asks the agent to reset it. The agent then tells the user that the password was reset to the standard new joiner password. The user does not have this password and asks the agent to call them to provide them with the password (solid security awareness!). The agent then commits an error and sends the password to the user directly via the portal. From here, we could try this password against the exposed VPN portal as the user may not have changed it.

Furthermore, the support agent states that this is the standard password given to new joiners and sets the user's password to this value. We have been in many organizations where the helpdesk uses a standard password for new users and password resets. Often the domain password policy is lax and does not force the user to change at the next login. If this is the case, it may work for other users. Though out of the scope of this module, in this scenario, it would be worth using tools like [linkedin2username](https://github.com/initstring/linkedin2username) to create a user list of company employees and attempt a password spraying attack against the VPN endpoint with this standard password.

Many applications such as osTicket also contain an address book. It would also be worth exporting all emails/usernames from the address book as part of our enumeration as they could also prove helpful in an attack such as password spraying.

#### Known Vulnerabilities
osTicket version 1.14.1 suffers from [CVE-2020-24881](https://nvd.nist.gov/vuln/detail/CVE-2020-24881) which was an SSRF vulnerability. If exploited, this type of flaw may be leveraged to gain access to internal resources or perform internal port scanning