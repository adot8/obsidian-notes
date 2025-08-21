
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

Notice how the filter criteria, `&`, `|`, and `!`, always appear at the beginning of the grouping, i.e. `(&(samAccountType=805306368)(adminCount=1))` rather than `((samAccountType=805306368)&(adminCount=1))`.

### Attributes
In all of these examples, the queries will return every attribute for the objects, which may be more information than we need.  Use the `--attributes` parameter in ldapsearch to control which attributes to return.  For example, if we only wanted the name and group memberships, we could do:

```powershell
beacon> ldapsearch (&(samAccountType=805306368)(adminCount=1)) --attributes name,memberof

--------------------
memberOf: CN=Group Policy Creator Owners,CN=Users,DC=contoso,DC=com, CN=Domain Admins,CN=Users,DC=contoso,DC=com, CN=Enterprise Admins,CN=Users,DC=contoso,DC=com, CN=Schema Admins,CN=Users,DC=contoso,DC=com, CN=Administrators,CN=Builtin,DC=contoso,DC=com
name: Administrator
--------------------
memberOf: CN=Denied RODC Password Replication Group,CN=Users,DC=contoso,DC=com
name: krbtgt
--------------------
memberOf: CN=Domain Admins,CN=Users,DC=contoso,DC=com
name: Dean York
--------------------
```

The ldapsearch BOF also allows you to read the security descriptor (i.e ACL) of an object by appending `ntsecuritydescriptor` to the list of attributes.  It's output as a long base64 string that BOFHound can parse and decode.

To return every attribute and the security descriptor, use `*,ntsecuritydescriptor`.

> Returning the security descriptor is essential if you want BloodHound to identify ACL-based attack paths.

#### Bitwise Filters

In addition to simple and's, or's, and not's, LDAP supports the use of bitwise operators by way of the following OIDs (object identifiers):

- 1.2.840.113556.1.4.803 for LDAP_MATCHING_RULE_BIT_AND.
    
- 1.2.840.113556.1.4.804 for LDAP_MATCHING_RULE_BIT_OR.

- 1.2.840.113556.1.4.1941 for LDAP_MATCHING_RULE_IN_CHAIN.

The bitwise AND is particularly useful for queries that target attributes such as [userAccountControl](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/useraccountcontrol-manipulate-account-properties).  This attribute is a set of bitwise flags, so performing a bitwise AND allows you to query whether a particular flag is set or not.  For example, TRUSTED_FOR_DELEGATION is a flag that gets set on a computer account when it's configured for [unconstrained delegation](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=6731fd18dc6d251272081c5f).  We could find every computer with this configuration by performing a bitwise AND with the value 524288.

```powershell
beacon> ldapsearch (&(samAccountType=805306369)(userAccountControl:1.2.840.113556.1.4.803:=524288)) --attributes samaccountname

sAMAccountName: LON-DC-1$
```

The LDAP_MATCHING_RULE_IN_CHAIN OID provides a method of querying the ancestry of an object, which becomes useful when needing to unroll groups of groups.  Consider a scenario where you query a domain group and find that another domain group is a member; you query that group and find two more groups as members.  Depending on the depth, querying each one manually can quickly get out of hand.  Not only is it laborious, it's prone to error and generates unnecessary LDAP traffic.

```powershell
beacon> ldapsearch "(memberof:1.2.840.113556.1.4.1941:=CN=Domain Admins,CN=Users,DC=contoso,DC=enclave)" --attributes samaccountname

--------------------
sAMAccountName: Administrator
--------------------
sAMAccountName: dyork
--------------------
sAMAccountName: Server Admins
--------------------
sAMAccountName: Workstation Admins
--------------------
sAMAccountName: Domain Users
```

The search above will return the ancestors of the Domain Admins group.  This example shows that all domain users are actually domain admins because it's nested in the Workstation Admins group, which is nested in the Server Admins group, which is nested in the Domain Admins group.

# _OPSEC Considerations_

In addition to the issue of signatured LDAP queries, a pain point even for custom data collection, is tripping a detection that looks for expensive, inefficient, or slow queries.  In each case, the trigger threshold is configurable by the organisation, so there are no definitive values and they will vary based on the size of the domain.

### Expensive Search Results Threshold
These are queries that return more results than the given threshold.  You could trigger these using extremely large pulls in a single query, like `(objectClass=*)`.

### Search Time Threshold
These are queries that take longer than N milliseconds to run.  The two main factors that contribute to the search time are its size (as above), and the number of attributes you've asked it to return.  Using `*,ntsecuritydescriptor` in ldapsearch will result in the slowest time for a given query.  If you're worried about this, dial it back to bare essentials like `samaccounttype,distinguishedname,objectsid,ntsecuritydescriptor`.

### Inefficient Search Results Threshold
These are queries that return less than 10% of the visited objects _if_ that number of visited objects is more than the given threshold.  Consider the following query which can be used to find [kerberoastable](https://www.zeropointsecurity.co.uk/path-player?courseid=red-team-ops&unit=67851a86ba9789af1107f5f9) users:  

```powershell
(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))
```

This will return user accounts whose SPN attribute is not null, whilst excluding the krbtgt account and disabled accounts.  You'd think that a query that returns fewer results would be considered more efficient, but that isn't the case.  This example will visit every user account in the domain, which could be hundreds or even thousands.  If fewer than 10% of those accounts have an SPN set, then this query would technically be inefficient because it will return less than 10% of the total visited objects.

In some cases, you may be better off enumerating the important details of every user in one go, like 

```powershell
ldapsearch (samAccountType=805306368) --attributes samaccounttype,distinguishedname,objectsid,serviceprincipalname,useraccountcontrol,ntsecuritydescriptor
```

This will return all of the visited objects (and therefore be considered efficient), and it will be reasonably quick if we keep the number of attributes down.  The only risk is if this tripped the expensive search results threshold.