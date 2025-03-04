
### Fuzz for subdomains & vhosts
```shell
ffuf -w ~/opt/wordlists/subdomains-top1million-110000.txt -u https://FUZZ.adot8.com
ffuf -H 'Host: FUZZ.cypher.htb' -w ~/opt/wordlists/subdomains_custom.txt:FUZZ -u http://cypher.htb
```
### Directory Fuzzing
```shell
ffuf  -w ~/opt/wordlists/web-extensions.txt -u http://cypher.htb/indexFUZZ

ffuf -w ~/opt/wordlists/directory-list-2.3-medium.txt -u http://cypher.htb/FUZZ 

ffuf -w ~/opt/wordlists/raft-medium-directories.txt -u http://cypher.htb/FUZZ 
```

While waiting for results perform on all pages:
1. Read source code manually + automated
	-  `python SecretFinder.py -i http://83.136.255.81`
2. Use ReconSpider
	-  `python3 ReconSpider.py https://adot8.com`
3. Deobfuscate any javascript

### Parameter Fuzzing (PHP)
```shell
ffuf -w ~/opt/wordlists/burp-parameter-names.txt -u http://shoppy.htb/login.php?FUZZ=1 -fs x

ffuf -w ~/opt/wordlists/burp-parameter-names.txt -u http://shoppy.htb/login.php -X POST -d 'FUZZ=1' -H 'Content-Type: application/x-www-form-urlencoded' -fs x
```

```bash
ffuf -w ~/opt/wordlists/sqli.txt -u http://shoppy.htb/login.php -X POST -d 'username=FUZZ&password=1' -H 'Content-Type: application/x-www-form-urlencoded' -fs x
```
Test parameters for the following
1. IDOR
2. File Inclusion
3. SQLi
4. XSS
5. Command Injection
### Search for input fields and test for the following
1. Command injection
2. SQL Injection
3. XXE
4. LDAP injecion
5. XSS

**TEST INJECTION PAYLOADS IN CONJUNCTION WITH [HTTP VERB TAMPERING](obsidian://open?vault=Penetration%20Testing&file=Root%2FWeb%20Applications%2FHTTP%20Verb%20Tampering%2FBypassing%20Security%20Filters)**

### Search for [File Upload](obsidian://open?vault=Penetration%20Testing&file=Root%2FWeb%20Applications%2FFile%20Upload%2F~%20Checklist) features
- Upload malicious SCF file (@hi.scf) and capture hash

### Search for Login Portals
1. Spray
```bash
admin:admin
root:admin
root:toor
administrator:admin
admin:Password123
```
2. Bypass with [HTTP Verb Tampering](obsidian://open?vault=Penetration%20Testing&file=Root%2FWeb%20Applications%2FHTTP%20Verb%20Tampering%2FBypassing%20Basic%20Authentication)
3. [Bruteforce](obsidian://open?vault=Penetration%20Testing&file=Root%2FPassword%20Attacks%2FLogin%20Brute%20Forcing%2FHydra%2FLogin%20Forms)