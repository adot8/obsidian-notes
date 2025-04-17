- Minimum of 8 characters.
- Include uppercase and lowercase letters.
- Include at least one number.
- Include at least one special character.
- It should not be the username.
- It should be changed every 60 days.

Based on this example, we must include, as part of our password policies, some blacklisted words, which include, but are not limited to:

- Company's name
- Common words associated with the company
- Names of months
- Names of seasons
- Variations on the word welcome and password
- Common and guessable words such as password, 123456, and abcde

## Enforcing Password Policy

A password policy is a guide that defines how we should create, manipulate and store passwords in the organization. To apply this guide, we need to enforce it, using the technology at our disposal or acquiring what needs to make this work. Most applications and identity managers provide methods to apply our password policy.

For example, if we use Active Directory for authentication, we need to configure an [Active Directory Password Policy GPO](https://activedirectorypro.com/how-to-configure-a-domain-password-policy/), to enforce our users to comply with our password policy.

Once the technical aspect is covered, we need to communicate the policy to the company and create processes and procedures to guarantee that our password policy is applied everywhere.

Creating a good password can be easy. Let's use [PasswordMonster](https://www.passwordmonster.com/), a website that helps us test how strong our passwords are, and [1Password Password Generator](https://1password.com/password-generator/), another website to generate secure passwords.

We can create good passwords with ordinary words, phrases, and even songs that we like. Here is an example of a good password `This is my secure password` or `The name of my dog is Poppy`. We can combine those passwords with special characters to make them more complex, like `()The name of my dog is Poppy!`. Although hard to guess, we should keep in mind that attackers can use OSINT to learn about us, and we should keep this in mind when creating passwords.

![Strong Password with a Phrase](https://academy.hackthebox.com/storage/modules/147/strong_password_phrase.png)

## Password Managers
A [password manager](https://en.wikipedia.org/wiki/Password_manager) is an application that allows users to store their passwords and secrets in an encrypted database. In addition to keeping our passwords and sensitive data safe, they also have features to generate and manage robust and unique passwords, 2FA, fill web forms, browser integration, synchronization between multiple devices, security alerts, among other features.

Most popular online password managers are:

1. [1Password](https://1password.com/)
2. [Bitwarden](https://bitwarden.com/)
3. [Dashlane](https://www.dashlane.com/)
4. [Keeper](https://www.keepersecurity.com/)
5. [Lastpass](https://www.lastpass.com/)
6. [NordPass](https://nordpass.com/)
7. [RoboForm](https://www.roboform.com/)

The most popular local password managers are:

1. [KeePass](https://keepass.info/)
2. [KWalletManager](https://apps.kde.org/kwalletmanager5/)
3. [Pleasant Password Server](https://pleasantpasswords.com/)
4. [Password Safe](https://pwsafe.org/)

 Alternatives
1. [Multi-factor Authentication](https://en.wikipedia.org/wiki/Multi-factor_authentication).
2. [FIDO2](https://fidoalliance.org/fido2/) open authentication standard, which enables users to leverage common devices like [Yubikey](https://www.yubico.com/), to authenticate easily. For a more extended device list, you can see [Microsoft FIDO2 security key providers](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless#fido2-security-key-providers).
3. [One-Time Password (OTP)](https://en.wikipedia.org/wiki/One-time_password).
4. [Time-based one-time password (TOTP)](https://en.wikipedia.org/wiki/Time-based_one-time_password).
5. [IP restriction](https://news.gandi.net/en/2019/05/using-ip-restriction-to-help-secure-your-account/).
6. Device Compliance. Examples: [Endpoint Manager](https://www.petervanderwoude.nl/post/tag/device-compliance/) or [Workspace ONE](https://www.loginconsultants.com/enabling-the-device-compliance-with-workspace-one-uem-authentication-policy-in-workspace-one-access)

Passwordless
Multiples companies like [Microsoft](https://www.microsoft.com/en-us), [Auth0](https://auth0.com/), [Okta](https://www.okta.com/), [Ping Identity](https://www.pingidentity.com/en.html), etc, are trying to promote the [Passwordless](https://en.wikipedia.org/wiki/Passwordless_authentication) strategy, to remove the password as the way of authentication.

[Passwordless](https://www.pingidentity.com/en/resources/blog/posts/2021/what-does-passwordless-really-mean.html) authentication is achieved when an authentication factor other than a password is used. A password is a knowledge factor, meaning it's something a user knows. The problem with relying on a knowledge factor alone is that it's vulnerable to theft, sharing, repeat use, misuse, and other risks. Passwordless authentication ultimately means no more passwords. Instead, it relies on a possession factor, something a user has, or an inherent factor, which a user is, to verify user identity with greater assurance