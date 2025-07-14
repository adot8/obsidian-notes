
LDAP is an industry standard protocol for accessing directory service information across a network.  It requires a client to establish a session with an LDAP server (i.e. domain controller) before it can submit queries to it - this would be on TCP port 389 for LDAP or 636 for LDAPS.  Sessions are typically authenticated using the current domain context of the caller or by using explicit domain credentials.

Each 'object' in a directory has an associated **objectClass** which defines the types of attributes it can have.  There are hundreds of classes, but some common ones include:

- #### computer
    Represents a computer account in the domain.
    
- #### domain
    Contains information about a domain.
    
- #### group
    Stores a list of user names, used to apply security principals to resources.
    
- #### person
    Contains information about a user.
    
- #### groupPolicyContainer
    A container for storing Group Policy Objects.
    
- #### organizationalUnit
    A container for storing users, computers, and other account objects.
    
- #### trustedDomain
    An object that represents a domain trust relationship.

### Querying LDAP

Annoyingly, some objects have an objectClass that you wouldn't necessarily expect, which can make them unsuitable as a basis for your search in some circumstances.  For example, computer accounts actually have the object classes of 'person', 'organizationalPerson', 'user', and 'computer'.  So searching for all 'person' or 'user' objects will actually return computers as well.

#### Indexes
As you typically find in most types of database, directory information is indexed by different fields to make searches more efficient.  Searching by an indexed field will be much faster than a non-indexed field.  Microsoft have [documented](https://learn.microsoft.com/en-us/windows/win32/adschema/attributes-indexed) which directory attributes are indexed, so we should use these where possible.  This can also be relevant to OPSEC which is discussed in more detail further down this page.

#### Filters
Using LDAP to search for objects requires you to provide a **filter** that defines the criteria of the search.  For example, to search for all user accounts, we could use the [sAMAccountType](https://learn.microsoft.com/en-us/windows/win32/adschema/a-samaccounttype) attribute to search for objects that have a type of `SAM_NORMAL_USER_ACCOUNT`.  This is 0x30000000 in hex, or 805306368 in decimal.

```powershell
beacon> ldapsearch (samAccountType=805306368)
```

A user object contains attributes used to describe the user such as their name, a description, domain SID, logon count, UAC flags, last logon time, group membership, and so on.  There are also generic attributes that appear on every object, regardless of their class, such as the distinguishedName and objectGUID.

```powershell
Binding to 10.10.120.1

[*] Distinguished name: DC=contoso,DC=com
[*] targeting DC: \\lon-dc-1.contoso.com
[*] Filter: (sAMAccountType=805306368)
[*] Scope of search value: 3
[*] Returning specific attribute(s): *

--------------------
objectClass: top, person, organizationalPerson, user
cn: Administrator
description: Built-in account for administering the computer/domain
distinguishedName: CN=Administrator,CN=Users,DC=contoso,DC=com
instanceType: 4
whenCreated: 20250124135010.0Z
whenChanged: 20250303132351.0Z
uSNCreated: 8196
memberOf: CN=Group Policy Creator Owners,CN=Users,DC=contoso,DC=com, CN=Domain Admins,CN=Users,DC=contoso,DC=com, CN=Enterprise Admins,CN=Users,DC=contoso,DC=com, CN=Schema Admins,CN=Users,DC=contoso,DC=com, CN=Administrators,CN=Builtin,DC=contoso,DC=com
uSNChanged: 36919
name: Administrator
objectGUID: f1f979c4-11a0-4929-8147-c53305544d1c
userAccountControl: 66048
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 0
lastLogoff: 0
lastLogon: 133861646754853919
logonHours: ////////////////////////////
pwdLastSet: 133821991894459748
primaryGroupID: 513
objectSid: S-1-5-21-3926355307-1661546229-813047887-500
adminCount: 1
accountExpires: 0
logonCount: 40
sAMAccountName: Administrator
sAMAccountType: 805306368
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=contoso,DC=com
isCriticalSystemObject: TRUE
dSCorePropagationData: 20250124140605.0Z, 20250124140605.0Z, 20250124135054.0Z, 16010101181216.0Z
lastLogonTimestamp: 133854817944673761
msDS-SupportedEncryptionTypes: 0
--------------------
```

A query can be tuned to return a smaller subset of results by including '_and_' (represented by `&`) and/or '_or_' (represented by `|`) criteria in the filter.  For example, the following returns every object that has a samAccountType of 805306368 _and_ whose adminCount attribute is set to 1.

```powershell
beacon> ldapsearch (&(samAccountType=805306368)(adminCount=1))
```

This next command returns every object that has a samAccountType of 805306368 _and_ whose description contains the string 'admin' _or_ whose samAccountName (i.e. username) contains the string 'adm'.

```powershell
beacon> ldapsearch (&(samAccountType=805306368)(|(description=*admin*)(samaccountname=*adm*)))
```

Objects can also be excluded by using a 'not' criteria (represented by `!`).  For example, we could tune the previous query to not return the krbtgt account:

```powershell
beacon> ldapsearch (&(samAccountType=805306368)(adminCount=1)(!(name=krbtgt)))
```