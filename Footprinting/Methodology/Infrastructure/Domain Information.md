First thing we should do is scrutinize the company's **`main website`**. Then, we should read through the texts, keeping in mind what technologies and structures are needed for these services

For example, many IT companies offer app development, IoT, hosting, data science, and IT security services, etc. If we encounter a service that we have had little to do with before, it makes sense and is necessary to get to grips with it and find out what activities it consists of and what opportunities are available. Those services also give us a good overview of how the company can be structured
### SSL Certificates
`SSL certificates` from the companies main website will likely include subdomains, uncovering additional assets
![[Pasted image 20241231143133.png]]

[crt.sh](https://crt.sh) can be used to find subdomains and their registry information

One liner for JSON output and unique subdomains
```bash
curl -s https://crt.sh/\?q\=octopitech.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

### Company Hosted Servers
Map subdomains to public IP addresses **(Important for staying in scope)**
```bash
for i in $(cat subdomains.lst);do host $i | grep "has address" | grep octopitech.com | cut -d" " -f1,4;done
```

### Shodan Query
```bash
for i in $(cat subdomain.list);do host $i | grep "has address" | grep octopitech.com | cut -d" " -f4 >> ip-addresses.txt;done

for i in $(cat ip-addresses.txt);do shodan host $i;done
```

### DNS Records
```shell
dig adot8.com <record type>
dig adot8.com ANY

whois adot8.com

dnsenum --enum adot8.com -r -f ~/opt/subdomains-top1million-110000.txt

gobuster vhost -u http://adot8.com -w ~/opt/subdomains-top1million-110000.txt --append-domain
gobuster dir -u http://adot8.com -w ~/opt/raft-medium-directories.txt -x txt,html -t 100
```

What we could see so far were entries on the DNS server, which at first glance did not look very interesting (except for the additional IP addresses). However, we could not see the third-party providers behind the entries shown at first glance

WE NEED TO LOOK AT EACH RECORD, VIEW THEM AS A SERVICE AND THEN THINK OF WAYS TO FURTHER ENUMERATE OR EXPLOIT THEM (in scope)

[Google Gmail](https://www.google.com/gmail/) indicates that Google is used for email management. Therefore, it can also suggest that we could access open GDrive folders or files with a link.

[Outlook](https://outlook.live.com/owa/) is another indicator for document management. Companies often use Office 365 with OneDrive and cloud resources such as Azure blob and file storage. Azure file storage can be very interesting because it works with the SMB protocol.

[LogMeIn](https://www.logmein.com/) is a central place that regulates and manages remote access on many different levels. However, the centralization of such operations is a double-edged sword. If access as an administrator to this platform is obtained (e.g., through password reuse), one also has complete access to all systems and information.