#### ../ Strip
One of the most basic filters against LFI is a search and replace filter, where it simply deletes substrings of (`../`) to avoid path traversals

Put double of that mf
```bash
....//....//....//....//....//etc/passwd
```

#### URL Encoding
Encode the entire payload instead. This can be done in Burp
```http
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
```

#### Approved Paths
Some web applications may also use Regular Expressions to ensure that the file being included is under a specific path. For example, the web application we have been dealing with may only accept paths that are under the `./languages` directory.

**This filter may be coupled with others**

We can bypass this >:)
```http
./languages/../../../../etc/passwd
./languages//....//....//....//....//....//etc/passwd
```

#### Appended Extension

As discussed in the previous section, some web applications append an extension to our input string (e.g. `.php`), to ensure that the file we include is in the expected extension. With modern versions of PHP, we may not be able to bypass this and will be restricted to only reading files in that extension, which may still be useful (**e.g. for reading source code**).

There are a couple of other techniques we may use, but they are `obsolete with modern versions of PHP and only work with PHP versions before 5.3/5.4`.
#### Path Truncation
In earlier versions of PHP, defined strings have a maximum length of 4096 characters, likely due to the limitation of 32-bit systems. If a longer string is passed, it will simply be `truncated`, and any characters after the maximum length will be ignored. Furthermore, PHP also used to remove trailing slashes and single dots in path names, so if we call (`/etc/passwd/.`) then the `/.` would also be truncated, and PHP would call (`/etc/passwd`). PHP, and Linux systems in general, also disregard multiple slashes in the path (e.g. `////etc/passwd` is the same as `/etc/passwd`). Similarly, a current directory shortcut (`.`) in the middle of the path would also be disregarded (e.g. `/etc/./passwd`).

If we combine both of these PHP limitations together, we can create very long strings that evaluate to a correct path. Whenever we reach the 4096 character limitation, the appended extension (`.php`) would be truncated, and we would have a path without an appended extension. Finally, it is also important to note that we would also need to `start the path with a non-existing directory` for this technique to work.

```bash
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

#### Null Bytes
PHP versions before 5.5 were vulnerable to `null byte injection`, which means that adding a null byte (`%00`) at the end of the string would terminate the string and not consider anything after it.
```http
/etc/passwd%00
```