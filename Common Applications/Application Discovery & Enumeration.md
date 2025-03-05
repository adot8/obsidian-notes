
Web Discovery
```bash
nmap -p 80,443,8000,8080,8180,8888,10000,4443 --open -oA web_discovery -iL scope_list
```
Using the XML output we can take screenshots of each web page with `Eyewitness`
```shell
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```
We can do the same with `aquatone` 
```shell
cat web_discovery.xml | aquatone -nmap
```
Whats running?
```bash
curl -s http://dev.inlanefreight.local/ | grep -E "Joomla|WordPress|Drupal"
```

```bash
admin:admin
admin:Welcome1
root:toor
```