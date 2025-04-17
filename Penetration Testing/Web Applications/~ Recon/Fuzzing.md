## Subdomain & Vhosts
```shell
ffuf -w ~/opt/subdomains-top1million-110000.txt -u https://FUZZ.adot8.com

ffuf -H 'Host: FUZZ.academy.htb' -w ~/opt/subdomains-top1million-5000.txt:FUZZ 
-u http://adot8.com:PORT/ 
```

> [!NOTE] Note
> Filter size after test run with `-fs`
## Directory & File
```shell
ffuf  -w ~/opt/web-extensions.txt -u http://adot8.com/indexFUZZ

ffuf -w ~/opt/directory-list-2.3-medium.txt -u http://adot8.com/FUZZ
```
## ## Parameters and Values
```shell
ffuf -w opt/burp-parameter-names.txt -u http://academy.htb:46804/admin.php FUZZ=1 -fs x

ffuf -w opt/burp-parameter-names.txt -u http://academy.htb:46804/admin.php -X POST -d 'FUZZ=1' -H 'Content-Type: application/x-www-form-urlencoded' -fs x

ffuf -w 1-1000.txt -u http://academy.htb:46804/admin.php -X POST -d 'id=1' -H 'Content-Type: application/x-www-form-urlencoded' -fs x

```


> [!NOTE] Note
> Create wordlist 1-1000 with for loop
> `for i in $(seq 1 1000); do echo $i >> ids.txt; done` 
