## Components  

There are three parts of Cobalt Strike that require some introduction.

- ### Beacon
    
    Beacon is Cobalt Strike's post-exploitation agent.  Once running on a compromised endpoint, it is responsible for communicating back to a team server, fetching post-exploitation jobs, executing them, and sending the results back.  Beacon itself is implemented as a Windows DLL but can be packaged into payloads of various formats including executables, PowerShell scripts and position independent shellcode.
    
- ### Team Server
    
    The team server is the central control and logging system.  It stores information collected during an engagement and builds up several data models that can be used to generate reports and throughout some Beacon workflows.
    
- ### Client  
    
    The client is used by the red team operators to connect to one or more team servers to manage the engagement and interact with Beacon payloads.  When connected to multiple team servers at the same time, the client collates and aggregates the data from them all to provide a single cohesive view of the engagement.

## Distributed operations

Cobalt Strike's design pattern for distributed operations is to stand up dedicated team servers for each phase of an engagement such as initial access, post-exploitation, and persistence.  The rationale being that if part of an operation is discovered and blocked, the other channels can be used to maintain access.  Cobalt Strike has multiple infrastructure consolidation features that make this easy to work with.
![[Pasted image 20250626114935.png]]When launching the client, you're prompted to enter the details of a team server that you'd like to connect to.

![[Pasted image 20250626115001.png]]

Once connected, you can connect to multiple other team servers at the same time.  The client will consolidate the data between them all such as listeners and active Beacon sessions.

![[Pasted image 20250626115020.png]]