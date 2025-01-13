```bash
snmp-check $ip -c public -v 2c
snmpwalk -c public -v1 $ip  
snmpwalk -c public -v2c $ip
snmpwalk -c public -v2c $ip 1.3.6.1.2.1.25.4.2.1.2
snmpwalk -c public -v2c $ip NET-SNMP-EXTEND-MIB::nsExtendOutputFull
snmpwalk -c public -v2c $ip | grep -v  _amd64
snmpwalk -c public -v2c $ip | grep -v  _all
snmpwalk -c public -v2c $ip > snmp.out


onesixtyone -c ~/opt/wordlists/snmp_community_strings.txt $ip
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt $ip
```

Bruteforce OID's
```
braa <community string>@<IP>:.1.3.6.* 
braa public@$ip:.1.3.6.*
```

`Simple Network Management Protocol` ([SNMP](https://datatracker.ietf.org/doc/html/rfc1157)) was created to monitor network devices. In addition, this protocol can also be used to handle configuration tasks and change settings remotely. SNMP-enabled hardware includes routers, switches, servers, IoT devices, and many other devices that can also be queried and controlled using this standard protocol.

In addition to the pure exchange of information, SNMP also transmits control commands using agents over UDP port `161`. The client can set specific values in the device and change options and settings with these commands. While in classical communication, it is always the client who actively requests information from the server, SNMP also enables the use of so-called `traps` over UDP port `162`.

### MIB
To ensure that SNMP access works across manufacturers and with different client-server combinations, the `Management Information Base` (`MIB`) was created. MIB is an independent format for storing device information. A MIB is a text file in which all queryable SNMP objects of a device are listed in a standardized tree hierarchy. It contains at least one `Object Identifier` (`OID`), which, in addition to the necessary unique address and a name, also provides information about the type, access rights, and a description of the respective object. MIB files are written in the `Abstract Syntax Notation One` (`ASN.1`) based ASCII text format. The MIBs do not contain data, but they explain where to find which information and what it looks like, which returns values for the specific OID, or which data type is used.
### OID
An OID represents a node in a hierarchical namespace. A sequence of numbers uniquely identifies each node, allowing the node's position in the tree to be determined. The longer the chain, the more specific the information. Many nodes in the OID tree contain nothing except references to those below them. The OIDs consist of integers and are usually concatenated by dot notation. We can look up many MIBs for the associated OIDs in the [Object Identifier Registry](https://www.alvestrand.no/objectid/).

| **Version** | **Description**                                                                                                                                                                                                                |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SNMPv1`    | SNMPv1 has `no built-in authentication` mechanism. Another main flaw of SNMPv1 is that it `does not support encryption`, meaning that all data is sent in plain text and can be easily intercepted                             |
| `SNMPv2`    | The version still exists today is `v2c`, and the extension `c` means community-based SNMP. The `community string` that provides security is only transmitted in plain text, meaning it has no built-in encryption              |
| `SNMPv3`    | Has `authentication` using username and password and transmission `encryption` (via `pre-shared key`) of the data. The complexity also increases to the same extent, with significantly more configuration options than `v2c`. |
### Default Configuration
The default configuration of the SNMP daemon defines the basic settings for the service, which include the IP addresses, ports, MIB, OIDs, authentication, and community strings.
```bash
adot8@htb[/htb]~ cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'

sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly
```
### Dangerous Settings
|**Settings**|**Description**|
|---|---|
|`rwuser noauth`|Provides access to the full OID tree without authentication.|
|`rwcommunity <community string> <IPv4 address>`|Provides access to the full OID tree regardless of where the requests were sent from.|
|`rwcommunity6 <community string> <IPv6 address>`|Same access as with `rwcommunity` with the difference of using IPv6.|