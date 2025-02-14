#### [GTFOBins](https://gtfobins.github.io)
#### SETUID
Files with `SUID` (Set User ID) permissions allows the file to ran with permissions of another specified user

> [!NOTE] Note
> 4 bits for READ, 2 bits for WRITE and 1 bit for EXECUTE. chmod 777 would be rwx across the board This [resource](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits) has more information about the `setuid` and `setgid` bits, including how to set the bits.

![[Pasted image 20250214072206.png]]

```bash
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

#### SETGID
Same thing but for groups
```bash
find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null
```