A great tool for password spraying **O365** is [TREVORspray](https://github.com/blacklanternsecurity/TREVORspray). Again, tools come and go so this isn't the only option

Spray valid emails with single password. 15 second delay to avoid detection
```bash
trevorspray -u valid_emails.txt -p 'Welcome123' --delay 15
```

Same thing but using AWS machines as proxies over SSH
``` bash
trevorspray -u valid_emails.txt -p 'Welcome123' --delay 5 --no-current-ip --ssh ubuntu@100.25.38.206 -k adot8.pem 
```
![[Pasted image 20250222152707.png]]

Sometimes you may get this warning instead of a **SUCCESS** notification. This warning can be a successful login as well. Verify using the actual **O365** login panel.
![](https://pnpt.adot8.com/~gitbook/image?url=https%3A%2F%2F152155081-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FUczSr73L34emqNMDOxDg%252Fuploads%252FnLlzG5cKY6w45Ty37FEf%252Fimage.png%3Falt%3Dmedia%26token%3D44cf64d6-09fb-4ec1-94e7-3bd9b7978507&width=768&dpr=4&quality=100&sign=4dfb1f73&sv=2)


> [!NOTE] **PLEASE**
> Identify the lockout policy before spraying so you don't lock everyone out of their accounts

If a company has 5 passwords attempts as their policy, you can try 4 passwords and wait an hour until the next round

Once we get a foothold into an account digging through **Outlook, Onedrive, OneNote, Sharepoint and Teams**

Don't spray against a VPN login portal because they have good detection systems in place.

### AWS Proxy Setup
We can setup a free **AWS Cloud** account and create **Ubuntu machines** using the **EC2** service and the **Free Tier Eligible** option for everything**.**

Use the default credentials and create a new key pair.

To spin up more you can go to your **Instances Console -> Actions -> Image and Templates -> Launch more like this.** Tie it to the same key pair and launch that sucker.