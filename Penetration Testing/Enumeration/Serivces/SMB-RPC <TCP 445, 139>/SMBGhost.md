https://github.com/Barriuso/SMBGhost_AutomateExploitation
https://www.exploit-db.com/exploits/48537

#### Initiation of the Attack

|**Step**|**SMBGhost**|**Concept of Attacks - Category**|
|---|---|---|
|`1.`|The client sends a request manipulated by the attacker to the SMB server.|`Source`|
|`2.`|The sent compressed packets are processed according to the negotiated protocol responses.|`Process`|
|`3.`|This process is performed with the system's privileges or at least with the privileges of an administrator.|`Privileges`|
|`4.`|The local process is used as the destination, which should process these compressed packets.|`Destination`|