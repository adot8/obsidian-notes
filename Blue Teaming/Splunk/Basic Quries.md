
Operators include: `=`, `!=`, `<`, `>`, `<=`, `>=`

Search within the `main`index for events that contain the term `UNKNOWN` anywhere in the event data.
```bash
index="main" "*UNKNOWN*"
```

##### Fields and comparison operators
Search within the `main` index for events that do `not` have an `EventCode` value of `1`.
```bash
index="main" EventCode!=1
```

##### Fields command
Retrieve all process creation events from the `main` index, the `fields` command excludes the `User` field from the search results. Thus, the results will contain all fields normally found in the [Sysmon Event ID 1](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=90001) logs, except for the user that initiated the process. Please note that utilizing `sourcetype` restricts the scope exclusively to `Sysmon` event logs.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | fields - User
```

##### Table command
`_time` is the timestamp of the event, `host` is the name of the host where the event occurred, and `Image` is the name of the executable file that represents the process.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | table _time, host, Image
```

##### Rename command
Rename the `Image` field to `Process` in the search results. `Image` field in Sysmon logs represents the name of the executable file for the process. By renaming it, all the subsequent references to `Process` would now refer to what was originally the `Image` field.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | rename Image as Process
```

##### Dedup command
The `dedup` command removes duplicate entries based on the `Image` field from the process creation events. This means if the same process (`Image`) is created multiple times, it will appear only once in the results.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | dedup Image
```

##### Sort command
Sort all process creation events in the `main` index in descending order of their timestamps (_time), i.e., the most recent events are shown first.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | sort - _time
```

##### Stats command
This query will return a table where each row represents a unique combination of a timestamp (`_time`) and a process (`Image`). The count column indicates the number of network connection events that occurred for that specific process at that specific time.

However, it's challenging to visualize this data over time for each process because the data for each process is interspersed throughout the table. We'd need to manually filter by process (`Image`) to see the counts over time for each one.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | stats count by _time, Image
```

##### Chart command
```bash

```