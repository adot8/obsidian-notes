Leaving default credentials and/or reusing the same password across technologies within internal networks is common practice among system and network administrators.

We can use the tool [DefaultCreds-Cheat-Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet) to search for default creds in their database or search `Google` for the technologies documentation and find them there
```bash
➤ creds search tomcat                                                                                                      
+----------------------------------+------------+------------+
| Product                          |  username  |  password  |
+----------------------------------+------------+------------+
| apache tomcat (web)              |   tomcat   |   tomcat   |
| apache tomcat (web)              |   admin    |   admin    |
+----------------------------------+------------+------------+
```

 [Default router credentials](https://www.softwaretestinghelp.com/default-router-username-and-password-list/)

We can use a platform like [Dehashed](https://dehashed.com) to find compromised credentials based off of domain names, emails, usernames, IP addresses, phone numbers, full names, etc.

```

```

If it's just password hashes, this can be tossed into [Hashes.com](https://hashes.com) as well.
#### Other resources
https://haveibeenpwned.com/
https://weleakinfo.io/
https://leakcheck.io/
https://snusbase.com/
https://scylla.sh/