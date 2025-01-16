#### Pre-made Hashcat Rules
```bash
/usr/share/hashcat/rules/custom.rule
/usr/share/hashcat/rules/best64.rule
/usr/share/hashcat/rules/rockyou-30000.rule
/usr/share/hashcat/rules/InsidePro-PasswordsPro.rule
```
#### Hashcat Rule Syntax
| **Function** | **Description**                                   |
| ------------ | ------------------------------------------------- |
| `:`          | Do nothing.                                       |
| `l`          | Lowercase all letters.                            |
| `u`          | Uppercase all letters.                            |
| `c`          | Capitalize the first letter and lowercase others. |
| `sXY`        | Replace all instances of X with Y.                |
| `$!`         | Add the exclamation character at the end.         |
#### Site Scraping
Scraping the site for specific lengths of words and applying the ruleset can be useful as well
```shell
cewl $url -d 4 -m 6 --lowercase -w company.wordlist
```
#### Rule Example
```bash
c $1 $!
c $1 $2 $3 $!
c $2 $0 $2 $4 $!
```

- c = capital
- $(character) = append character
- u = uppercase
- d = duplicate

```bash
$ hashcat -r new.rule --stdout seasons.txt
January2024!
February2024!
March2024!
April2024!
...
```

```bash
:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

```bash
$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
$ cat mut_password.list
password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
Password!
passw0rd!
p@ssword!
Passw0rd!
P@ssword!
p@ssw0rd!
P@ssw0rd!
```