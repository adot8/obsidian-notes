### Footprinting + Enumeration
```bash
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

The web.xml configuration above defines a new servlet named AdminServlet that is mapped to the class com.inlanefreight.api.AdminServlet. Java uses the dot notation to create package names, meaning the path on disk for the class defined above would be:

    classes/com/inlanefreight/api/AdminServlet.class

Next, a new servlet mapping is created to map requests to /admin with AdminServlet. This configuration will send any request received for /admin to the AdminServlet.class class for processing. The web.xml descriptor holds a lot of sensitive information and is an important file to check when leveraging a Local File Inclusion (LFI) vulnerability.

The tomcat-users.xml file is used to allow or disallow access to the /manager and host-manager admin pages.

```bash
!-- user manager can access only manager section -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- user admin can access manager and admin section both -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />
```
