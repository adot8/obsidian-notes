Fuzz for subdomains & vhosts
```shell
ffuf -w ~/opt/wordlists/subdomains-top1million-110000.txt -u https://FUZZ.adot8.com
ffuf -H 'Host: FUZZ.adot8.com' -w ~/opt/wordlists/subdomains-top1million-5000.txt:FUZZ -u http://adot8.com 
```
Directory Fuzzing
```shell
ffuf  -w ~/opt/web-extensions.txt -u http://adot8.com/indexFUZZ
ffuf -w ~/opt/wordlists/directory-list-2.3-medium.txt -u http://adot8.com/FUZZ -e .php,.phps
```
While waiting for results perform on all pages:
1. Read source code manually + automated
	-  `python SecretFinder.py -i http://83.136.255.81`
2. Use ReconSpider
	-  `python3 ReconSpider.py https://adot8.com`
3. Deobfuscate any javascript
Parameter Fuzzing
```shell
ffuf -w opt/burp-parameter-names.txt -u http://academy.htb:46804/admin.php FUZZ=1 -fs x

ffuf -w opt/burp-parameter-names.txt -u http://academy.htb:46804/admin.php -X POST -d 'FUZZ=1' -H 'Content-Type: application/x-www-form-urlencoded' -fs x
```
Search for input fields and test for the following
- XSS
- SQLi
- Command injection
- SSRF