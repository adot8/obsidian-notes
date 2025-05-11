**Essentially an ACL baseline that overwrites any changes made to protected groups**

Resides in the System container of a domain and used to control the permissions - using an ACL - for certain built-in privileged groups (called Protected Groups)

Security Descriptor Propagator (`SDPROP`) runs every hour and compares the ACL of protected groups and members with the ACL of `AdminSDHolder` and any differences are overwritten on the object ACL

Protected Groups:

| Account Operators | Enterprise Admins            |
| ----------------- | ---------------------------- |
| Backup Operators  | Domain Controllers           |
| Server Operators  | Read-only Domain Controllers |
| Print Operators   | Schema Admins                |
| Domain Admins     | Administrators               |
| Replicator        |                              |

Well known abuse of some of the Protected Groups - All of the below
can log on locally to DC:

| Group             | Abuse                                                                                |
| ----------------- | ------------------------------------------------------------------------------------ |
| Account Operators | Cannot modify DA/EA/BA groups. Can modify nested group within these groups.          |
| Backup Operators  | Backup GPO, edit to add SID of controlled account to a privileged group and Restore. |
| Server Operators  | Run a command as system (using the disabled Browser service)                         |
| Print Operators   | Copy ntds.dit backup, load device drivers                                            |

With DA privileges (Full Control/Write permissions) on the
`AdminSDHolder` object, it can be used as a backdoor/persistence
mechanism by adding a user with Full Permissions (or other interesting
permissions) to the `AdminSDHolder` object

In 60 minutes (when `SDPROP` runs), the user will be added with Full
Control to the AC of groups like Domain Admins without actually being a
member of it