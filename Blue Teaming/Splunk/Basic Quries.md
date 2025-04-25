
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
This query will return a table where each row represents a unique timestamp (`_time`) and each column represents a unique process (`Image`). The cell values indicate the number of network connection events that occurred for each process at each specific time.

With the `chart` command, you can easily visualize the data over time for each process because each process has its own column. You can quickly see at a glance the count of network connection events over time for each process.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 | chart count by _time, Image
```

##### Eval command
This command creates a new field `Process_Path` which contains the lowercase version of the `Image` field. It doesn't change the actual `Image` field, but creates a new field that can be used in subsequent operations or for display purposes.
```bash
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | eval Process_Path=lower(Image)
```

##### Rex command
```bash
index="main" EventCode=4662 | rex max_match=0 "[^%](?<guid>{.*})" | table guid
```
- `index="main" EventCode=4662` filters the events to those in the `main` index with the `EventCode` equal to `4662`. This narrows down the search to specific events with the specified EventCode.
- `rex max_match=0 "[^%](?<guid>{.*})"` uses the rex command to extract values matching the pattern from the events' fields. The regex pattern `{.*}` looks for substrings that begin with `{` and end with `}`. The `[^%]` part ensures that the match does not begin with a `%` character. The captured value within the curly braces is assigned to the named capture group `guid`.
- `table guid` displays the extracted GUIDs in the output. This command is used to format the results and display only the `guid` field.
- The `max_match=0` option ensures that all occurrences of the pattern are extracted from each event. By default, the rex command only extracts the first occurrence.

This is useful because GUIDs are not automatically extracted from 4662 event logs.

##### Lookup commnad
The `lookup` command enriches the data with external sources. Example:

Suppose the following CSV file called `malware_lookup.csv`.

```shell-session
filename, is_malware
notepad.exe, false
cmd.exe, false
powershell.exe, false
sharphound.exe, true
randomfile.exe, true
```

This CSV file should be added as a new Lookup table as follows.

![Splunk interface showing the Search page with a Settings menu open. Options include Lookups, Data inputs, and Indexes.](https://academy.hackthebox.com/storage/modules/218/107.png)

![Splunk Lookups page with options to manage Lookup table files, Lookup definitions, Automatic lookups, and GeoIP lookups file.](https://academy.hackthebox.com/storage/modules/218/108.png)

![Splunk Lookup table files page showing a list of CSV files with paths, owners, and sharing settings. Includes a button for 'New Lookup Table File'.](https://academy.hackthebox.com/storage/modules/218/109.png)

![Splunk interface for adding a new lookup table file. Destination app is 'search', uploading 'malware_lookup.csv', with the destination filename 'malware_lookup.csv'.](https://academy.hackthebox.com/storage/modules/218/110.png)

```shell-session
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 | rex field=Image "(?P<filename>[^\\\]+)$" | eval filename=lower(filename) | lookup malware_lookup.csv filename OUTPUTNEW is_malware | table filename, is_malware
```

- `index="main" sourcetype="WinEventLog:Sysmon" EventCode=1`: This is the search criteria. It's looking for Sysmon logs (as identified by the sourcetype) with an EventCode of 1 (which represents process creation events) in the "main" index.
- `| rex field=Image "(?P<filename>[^\\\]+)$"`: This command is using the regular expression (regex) to extract a part of the Image field. The Image field in Sysmon EventCode=1 logs typically contains the full file path of the process. This regex is saying: Capture everything after the last backslash (which should be the filename itself) and save it as filename.
- `| eval filename=lower(filename)`: This command is taking the filename that was just extracted and converting it to lowercase. The lower() function is used to ensure the search is case-insensitive.
- `| lookup malware_lookup.csv filename OUTPUTNEW is_malware`: This command is performing a lookup operation using the filename as a key. The lookup table (malware_lookup.csv) is expected to contain a list of filenames of known malicious executables. If a match is found in the lookup table, the new field is_malware is added to the event, which indicates whether or not the process is considered malicious based on the lookup table. `<-- filename in this part of the query is the first column title in the CSV`.
- `| table filename, is_malware`: This command is formatting the output to show only the fields filename and is_malware. If is_malware is not present in a row, it means that no match was found in the lookup table for that filename.

Basically, this query is extracting the filenames of newly created processes, converting them to lowercase, comparing them against a list of known malicious filenames, and presenting the findings in a table.