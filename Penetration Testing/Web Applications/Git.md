Located at
```bash
http://gitea.site.htb
http://site.htb/.git
http://site.htb/dev/.git
```

Pull source code from .git directory
```bash
git-dumper http://site.htb/.git website
```

View all commits
```bash
git log
git show <commit id>
```
 > Check .**htaccess** or **.htpasswd**
 
 Check a commit
```bash
cp -r website git
git checkout '8812785e31c879261050e72e20f298ae8c43b565'
```
