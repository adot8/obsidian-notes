Change the user agent to blend in

```powershell
export AGENT="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,like Gecko) Chrome/87.0.4280.88 Safari/537.36"
```

```powershell
alias curl="curl -A '$AGENT'"

alias wget="wget -U '$AGENT'"

alias nmap="nmap --script-args=\"http.useragent='$AGENT'\""

alias wpscan="wpscan --ua '$AGENT'"
```

