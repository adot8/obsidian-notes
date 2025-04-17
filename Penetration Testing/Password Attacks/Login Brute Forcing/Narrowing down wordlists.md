Password Policy:
- Minimum length: 8 characters
- Must include:
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one number
```shell
grep -E '^.{8,}$' darkweb2017-top10000.txt > darkweb2017-minlength.txt

grep -E '[A-Z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt

grep -E '[a-z]' darkweb2017-minlength.txt > darkweb2017-uppercase.txt

grep -E '[0-9]' darkweb2017-lowercase.txt > darkweb2017-number.txt

mv darkweb2017-number.txt darkweb2017-filtered.txt
```

Password Policy:
- Minimum length: 6 characters
- Must include:
    - At least one uppercase letter
    - At least one lowercase letter
    - At least one number
    - 2 special characters

```shell
grep -E '^.{6,}$' jane.txt | grep -E '[A-Z]' | grep -E '[a-z]' | grep -E '[0-9]' | grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt
```