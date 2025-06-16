### Splunk Instances
- Splunk can be configured on a `physical machine`, `VM` or `container`
- Instances can be configured to have **one or more** the following `processing` and `management` components:

| Processing   | Management              |
| ------------ | ----------------------- |
| Indexers     | Deployment server       |
| Forwarders   | Indexer Cluster Manager |
| Search Heads | License Manager         |
|              | Monitoring Console      |

![[Pasted image 20250616093121.png]]

### Forwarders
- Forwarders are installed on host machines to collect and send data to Splunk for indexing
- This is the primary way to send logs to Splunk
- There are two types of forwarders:

| Universal Forwarder                            | Heavy Forwarder                                                                            |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Seperate executable with reduced functionality | Create a Splunk entrerprise instance and then configure it to be a `Heavy Forwarder`       |
| Main function is to collect and send data      | Can parse and make changes to data before forwarding it to another Splunk instance indexer |
| Downloadable from Splunk site                  |                                                                                            |


> [!NOTE] Practice Questions
> **Which of the following Splunk components typically resides on machines where data originates?**
> `Forwarder`
> **Which of the following Splunk components can perform log filtering/parsing?**
> `Heavy Forwarder`

![[Pasted image 20250616094205.png]]
### Indexers
- The component that receives the data from the `forwarders` and host machines is called an `Indexer`
- The `Indexer` can parse data and scrub metadata from the data/logs. 
	- For example it can scrub the `sourcetype` of the data to distinguish what kind of data it is
	- It can also assign timestamps to the logs as well

##### Indexes
- After the `Indexer` processes the data it then writes it to **disk** into small repositories called `indexes`
- Some examples of `indexes` on Disk include:
	- `main`
	- `_internal` (these are meant for Splunks own internal logs)
	- Custom indexes

##### Buckets
- These `indexes` are stored into repositories called `Buckets`which holds the files with the indexed data 
- These `Buckets` are found in the Home Path
- There are two types of buckets

| Hot Bucket                                                                      | Warm Bucket |
| ------------------------------------------------------------------------------- | ----------- |
| Used as the bucket you are **ACTIVELY** writing data into as it's being indexed |             |
