
> [!NOTE] Note
> Legit just read through front end source code with view source to try and find credentials, hidden directories, hidden subdomains, hidden functions, etc.

This can be automated with a tool like [SecretFinder](https://github.com/m4ll0k/SecretFinder) and custom regex wordlists.
```python
    'possible_Creds' : r"(?i)(" \
                    r"password\s*[`=:\"]+\s*[^\s]+|" \
                    r"password is\s*[`=:\"]*\s*[^\s]+|" \
                    r"pwd\s*[`=:\"]*\s*[^\s]+|" \
                    r"admin\s*[`=:\"]*\s*[^\s]+|"\
                    r"administrator\s*[`=:\"]*\s*[^\s]+|" \
                    r"dev\s*[`=:\"]*\s*[^\s]+|" \
                    r"root\s*[`=:\"]*\s*[^\s]+|" \
                    r"cred\s*[`=:\"]*\s*[^\s]+|" \
                    r"creds\s*[`=:\"]*\s*[^\s]+|" \
                    r"passwd\s*[`=:\"]+\s*[^\s]+)",
}
```

```shell
python SecretFinder.py -i http://83.136.255.81
```

