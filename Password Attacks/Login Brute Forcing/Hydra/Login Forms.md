Success and failure conditions are crucial for properly identifying valid and invalid login attempts. Hydra primarily relies on failure conditions (`F=...`) to determine when a login attempt has failed, but you can also specify a success condition (`S=...`) to indicate when a login is successful.

The failure condition (`F=...`) is used to check for a specific string in the server's response that signals a failed login attempt. Can be a error message like `Invalid Credentials`
```shell
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=Dashboard"
```

If the application redirects the user after a successful login (using HTTP status code `302`), or displays specific content (like "Dashboard" or "Welcome"), you can configure Hydra to look for that success condition using `S=`
```shell
hydra ... http-post-form "/login:user=^USER^&pass=^PASS^:S=302"
```

- The form submits data to the root path (`/`).
- The username field is named `username`.
- The password field is named `password`.
- An error message "Invalid credentials" is displayed upon failed login.
```shell
/:username=^USER^&password=^PASS^:F=Invalid credentials
```