#### Footprinting + Enumeration
```bash
wpscan --url $url 
wpscan --url $url -e vp     <- Vulnerable plugins
wpscan --url $url -e cb     <- Config backups
wpscan --url $url -e p --plugins-detection aggressive
```

```bash
curl -s http://blog.inlanefreight.local | grep WordPress
```

```bash
curl http://inlanefreight.local/robots.txt

User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://blog.inlanefreight.local/wp-sitemap.xml
```

WordPress stores its plugins in the `wp-content/plugins` directory. This folder is helpful to enumerate vulnerable plugins. Themes are stored in the `wp-content/themes` directory. These files should be carefully enumerated as they may lead to RCE.

Discover installed/used themes and plugins
```bash
curl -s http://blog.inlanefreight.local/ | grep themes
curl -s http://blog.inlanefreight.local/ | grep plugins
```
Visiting `/wp-content/plugins/realplugin` can uncover `readme`'s with the version

We can discover users by attempting to login via `/wp-admin` and the error messages they produce