```shell
./finalrecon --full --url http://ado8.com 
```

## Manual
```shell
whois adot8.com

dig adot8.com <record type>
dig adot8.com ANY

dnsenum --enum adot8.com -r -f ~/opt/subdomains-top1million-110000.txt
gobuster vhost -u http://adot8.com -w ~/opt/subdomains-top1million-110000.txt --append-domain
gobuster dir -u http://adot8.com -w ~/opt/raft-medium-directories.txt -x txt,html -t 100

curl -s "https://crt.sh/?q=adot8.com.com&output=json" | jq -r '.[]
| select(.name_value | contains("dev")) | .name_value' | sort -u
```

> [!NOTE] Title
> Repeat the the subdomain enumeration for each subdomain as they can be nested. This includes directory busting, web crawling and further vhost brute forcing.

```shell
curl -I adot8.com

wafw00f adot8.com

nikto -h adot8.com -Tuning b

python3 ReconSpider.py https://adot8.com
```

## Search Engine Operators
- Finding Login Pages:
	- `site:example.com inurl:login`
    - `site:example.com (inurl:login OR inurl:admin)`
- Identifying Exposed Files:
    - `site:example.com filetype:pdf`
    - `site:example.com (filetype:xls OR filetype:docx)`
- Uncovering Configuration Files:
    - `site:example.com inurl:config.php`
    - `site:example.com (ext:conf OR ext:cnf)` (searches for extensions commonly used for configuration files)
- Locating Database Backups:
    - `site:example.com inurl:backup`
    - `site:example.com filetype:sql`