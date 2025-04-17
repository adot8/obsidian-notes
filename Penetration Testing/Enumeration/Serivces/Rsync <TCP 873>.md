##### Scanning
```bash
nmap -sV -p 873 $ip
```
##### Probe Shares
```bash
nc -nv $ip 873
```
##### Enumerate Open Share
```bash
rsync -av --list-only rsync://$ip/share
```

##### Sync Files (Download)
```bash
rsync -av rsync://$ip/share
```
[Rsync](https://linux.die.net/man/1/rsync) is a fast and efficient tool for locally and remotely copying files. It can be used to copy files locally on a given machine and to/from remote hosts. It is highly versatile and well-known for its delta-transfer algorithm. This algorithm reduces the amount of data transmitted over the network when a version of the file already exists on the destination host. It does this by sending only the differences between the source files and the older version of the files that reside on the destination server. It is often used for backups and mirroring. It finds files that need to be transferred by looking at files that have changed in size or the last modified time. By default, it uses port `873` and can be configured to use SSH for secure file transfers by piggybacking on top of an established SSH server connection.

This [guide](https://book.hacktricks.xyz/network-services-pentesting/873-pentesting-rsync) covers some of the ways Rsync can be abused, most notably by listing the contents of a shared folder on a target server and retrieving files.