---
{"dg-publish":true,"permalink":"/cards/blue-team/endpoint-security/sysmon/"}
---

[[map-of-contents/blue-team\|blue-team]]

- Exclusion is better than inclusion.

**Event IDs**
- 1 - process creation.
	- **process guid** is always unique unlike PID can be reused.
- 3 - network connection. (disabled by default)
	- each connection has it's own PID and PGUID.
- 5 - process terminated. (nosiy need to tune out MS activities.)
- 7 - Image loaded - such as processes using `.dll`
- 8 - CreateRemoteThread
	- One process is injecting and running code in another process's space.
- 10 - ProcessAccess.
	- Process request access or opens another process.
- 11 - FileCreate (or Overwritten).
- 12, 13, 14 - Registry Events
	- 12 - creation and deletion.
	- 13 - registry value modification.
	- 14 - any key or value renamed.
- 15 - FileCreateStreamHash
	- the creation or modification of Alternate Data Streams (ADS) - _mark of the web_ is basically the tell tale for browser whether a binary is malicious or not being downloaded
- 22 - A process made a DNS query.
	- **Best practice:** exclude all known domains by checking proxy or firewall logs to determine top domain visited
- 29 - file executable detected

Query via ProcessId in Event Viewer XML:
```Powershell
<QueryList> 
 <Query Id="0" Path="Microsoft-Windows-Sysmon/Operational">
  <Select Path="Microsoft-Windows-Sysmon/Operational"> *[System[Provider[@Name='Microsoft-Windows-Sysmon']]] and *[EventData[Data[@Name='ProcessId'] and (Data='<ENTER YOUR PID HERE>')]] </Select>
 </Query>
</QueryList>
```

Query via Process Creation Events (1):

```Powershell
<QueryList>
  <Query Id="0" Path="Microsoft-Windows-Sysmon/Operational">
	<Select Path="Microsoft-Windows-Sysmon/Operational">
  	*[System[Provider[@Name='Microsoft-Windows-Sysmon'] and (EventID=1)]]
  	and
  	*[EventData[Data[@Name='ProcessId'] and (Data='<ENTER YOUR PID HERE>')]]
	</Select>
  </Query>
</QueryList>
```

## Headless Query
---

**Query with max events and all columns** (or data.)
```powershell
Get-WinEvent -FilterHashtable @{logname="Microsoft-Windows-Sysmon/Operational"; id=3} -MaxEvents 1 | Format-List *
```

**Query specific fields and EventIDs**

```Powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[System/EventID=3 and EventData[Data[@Name='DestinationPort']='4444']]" | Format-List *
```

**Pull back process ID**:

```Powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' -FilterXPath "*[System/EventID=1 and EventData[Data[@Name='ProcessId']='<ENTER YOUR PID HERE>']]" | Format-List *
```