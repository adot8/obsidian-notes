[PATH](http://www.linfo.org/path_env_var.html) is an environment variable that specifies the set of directories where an executable can be located. An account's PATH variable is a set of absolute paths, allowing a user to type a command without specifying the absolute path to the binary. For example, a user can type `cat /tmp/test.txt` instead of specifying the absolute path `/bin/cat /tmp/test.txt`.

```bash
htb_student@NIX02:~ echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
```

Adding `.` to a user's PATH adds their current working directory to the list. For example, if we can modify a user's path, we could replace a common binary such as `ls` with a malicious script such as a reverse shell. If we add `.` to the path by issuing the command `PATH=.:$PATH` and then `export PATH`, we will be able to run binaries located in our current working directory by just typing the name of the file (i.e. just typing `ls` will call the malicious script named `ls` in the current working directory instead of the binary located at `/bin/ls`).

```bash
PATH=.:${PATH}
export PATH
echo $PATH
```

In this example, we modify the path to run a simple `echo` command when the command `ls` is typed.

```bash
touch ls
echo 'echo "PATH ABUSE!!"' > ls
chmod +x ls
```