### Identify

In the `File Manager` web application, if we try to create a new file name with special characters in its name (e.g. `test;`), we get the following message:

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_malicious_request.jpg)

This message shows that the web application uses certain filters on the back-end to identify injection attempts and then blocks any malicious requests. No matter what we try, the web application properly blocks our requests and is secured against injection attempts. However, we may try an HTTP Verb Tampering attack to see if we can bypass the security filter altogether.

---
### Exploit

To try and exploit this vulnerability, let's intercept the request in Burp Suite (Burp) and then use `Change Request Method` to change it to another method: ![unauthorized_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_GET_request.jpg)

This time, we did not get the `Malicious Request Denied!` message, and our file was successfully created:

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_injected_request.jpg)

To confirm whether we bypassed the security filter, we need to attempt exploiting the vulnerability the filter is protecting: a Command Injection vulnerability, in this case. So, we can inject a command that creates two files and then check whether both files were created. To do so, we will use the following file name in our attack (`file1; touch file2;`):

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_filter_bypass.jpg)

Then, we can once again change the request method to a `GET` request: ![filter_bypass_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_filter_bypass_request.jpg)

Once we send our request, we see that this time both `file1` and `file2` were created:

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_after_filter_bypass.jpg)

