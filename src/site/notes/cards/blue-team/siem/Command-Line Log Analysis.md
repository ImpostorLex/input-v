---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/command-line-log-analysis/"}
---

~ [[atlas/blue-team\|blue-team]]

- No agent installed on the endpoint or handed a log file scenario.
- `less` is good.
- `grep -n` add line number

 understand the format of log:
```bash
file access.log
```

How many events and size we are dealing with?
```bash
ls -lh access.log && wc access.log
```

Ensure that these logs are what we want or from the time of the incident:
```
tail -n 15 & head -n 15
```

sample log:
```C
91.15.183.209 - - [17/Jul/2024:18:47:37 -0400] "GET /users/02492184 HTTP/1.1" 200 200 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36"
```

extracting specific columns (the '-' will act as a column):
```bash
cut -d " " -f 1
```

- `awk` is good see below.

additional formatting:
```bash
| sort | uniq -c | sort -nr
```

- `grep -v " 1 " | ...` add before  `sort -nr` to remove only one occurence.

extracting user agent strings (Since it is enclosed with double quotes):
```bash
cut -d "\"" -f 6 access.log
```

**note**: (delimeter will stop at first appearance)

![Command-Line Log Analysis.png](/img/user/cards/blue-team/endpoint-security/images/Command-Line%20Log%20Analysis.png)
Output:

![Command-Line Log Analysis-1.png](/img/user/cards/blue-team/endpoint-security/images/Command-Line%20Log%20Analysis-1.png)
Then we can use `grep` to output all events relating to a specific user agent:

```bash
grep "Nmap Scripting Engine" or grep "Mozilla/5.0 (Hydra)"
```

Extracting columns:
```bash
grep "Mozilla/5.0 (Hydra)" access.log | awk '{print $1}'
```

**Investigating a brute-force**
- We can rely on the size parameter as incorrect logon usually displays small messsage but for success it is typical a redirection to a much larger site.
- or via status code: 200 if it's incorrect or a GET request but 301 for redirection which typically means succesful login:

```bash
grep "Mozilla/5.0 (Hydra)" access.log | awk '$9 > 200'
```

Output:
![Command-Line Log Analysis-2.png](/img/user/cards/blue-team/endpoint-security/images/Command-Line%20Log%20Analysis-2.png)
## Pattern Matching
---
**Grep**
- `-n` add a line number
- `-c` count

Detecting cross site scripting (refer to url encoded)
```bash
grep -E '%3C|%3E|<|>'
```

