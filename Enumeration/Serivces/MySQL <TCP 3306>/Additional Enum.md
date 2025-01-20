#### Compromised Windows machine
```powershell
mysql.exe -u username -pPassword123 -h 10.129.20.13
```

```powershell
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```
> [!NOTE] Note
> When we authenticate to MSSQL using `sqlcmd` we can use the parameters `-y` (SQLCMDMAXVARTYPEWIDTH) and `-Y` (SQLCMDMAXFIXEDTYPEWIDTH) for better looking output. Keep in mind it may affect performance.



### GUI Applications
[dbeaver](https://github.com/dbeaver/dbeaver) is a multi-platform database tool for Linux, macOS, and Windows that supports connecting to multiple database engines such as MSSQL, MySQL, PostgreSQL, among others, making it easy for us, as an attacker, to interact with common database servers.

To install [dbeaver](https://github.com/dbeaver/dbeaver) using a Debian package we can download the release .deb package from [https://github.com/dbeaver/dbeaver/releases](https://github.com/dbeaver/dbeaver/releases) and execute the following command:

```shell
sudo dpkg -i dbeaver-<version>.deb
```

https://www.youtube.com/watch?v=PeuWmz8S6G8

```bash
dbeaver &
```
![[Pasted image 20250115140700.png]]