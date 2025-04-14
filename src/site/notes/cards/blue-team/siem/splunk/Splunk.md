---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/splunk/splunk/","tags":["splunk"]}
---

[[cards/blue-team/siem/Splunk Map\|Splunk Map]]
## Investigations

**Determine what are the available indexes and their count:**
```C
| eventcount summarize=false index=*
```

**Revealing all source types:**
```C
| metadata type=sourcetypes
| fields sourcetype
```

- **See the top talker:** summarize the count of IP addresses talking to our endpoint. preferrably use our endpoint ip as `ip_dest`.

```bash
index=* | stats count by src_ip | sort -count
```

- **Reduce traffic:** make use of the `sourcetype` found.
- **Keywords:** sometimes a keyword is all you need.
- **Move on:** if the question is too difficult gather more evidence.
#### Basics
---
- Installation: best practice is in linux and at the `/opt` directory (optional softwares)
- Binaries located: `/opt/splunk/bin`

Setup:
```bash
sudo ./splunk start --accept-license #admin:password
```

- **index** a repository where all data is stored and organized, think about it as a database.

---
- `Save As`: using a specific search query, we can do:
	- **Alert** 
	- **Report**
	- **Add to dashboard**

### Syntax
---
- Add double quotes to any words we want to match.
	- `AND` operator is applied hiddenly when multiple conditions is inputted.
- `*` match that startswith or endswith.
	- `log*n` = login or logon.
- **not equal** `file != login.php`.
- `status>200` or `>=404`, `<=404`.
	- - [Web HTTP Status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- `NOT`, `OR`, `AND` = Order of evaluation keep in mind
- `earliest=-1y` and `latest=-1w` can use absolute time (YYYY-MM-DD:MM:SS)

##### Search Commands
---
- https://docs.splunk.com/Documentation/Splunk/9.2.2/SearchReference/ListOfSearchCommands

View all unique sourcetypes:
```C
| metadata type=sourcetypes
| fields sourcetypes
```

- `search1 | command1` pass data to another command, the pipe
	- Add the `| search <keyword>` to enable searching against the filtered data (known as subsearch)
- `sort req_time` via field.
	- Show the latest first: `| sort -req_time`

Sample usage:
![Splunk.png](/img/user/cards/blue-team/siem/images/Splunk.png)
- `head 5` shows top 5 results or `tail 5` show top 5 last results.
- `table`

![Splunk-1.png](/img/user/cards/blue-team/siem/images/Splunk-1.png)
- `| dedup useragent` remove duplicate useragents
- `| top limit=5 useragent` shows top 5 most occured user agent or opposite `rare`.

![Splunk-2.png](/img/user/cards/blue-team/siem/images/Splunk-2.png)
Visualize the attack:

![Splunk-3.png](/img/user/cards/blue-team/siem/images/Splunk-3.png)

- `| iplocation clientip` lookup ip address
	- `| geostats count by Country` then visualization.

## Dashboard

- List out all top user-agents seen and URI path requested.
- IP addresses by geolocation.
- HTTP connection Over time



