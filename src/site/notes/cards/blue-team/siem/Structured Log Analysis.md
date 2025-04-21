---
{"dg-publish":true,"permalink":"/cards/blue-team/siem/structured-log-analysis/"}
---

[[map-of-contents/blue-team\|blue-team]]

File references: **events.json**.

Count the number of objects inside this array:
```bash
jq 'length' events.json
```

Since we are dealing with a `JSON` as an `array`: (Starts with `[]` ): we need to access or get 'inside' the array by:

```bash
jq '.[]' events.json
```

- `.` used to access an object properties
- `[]` required for accessing an array.


```bash
jq '.[] | select(.event.PROCESS_ID == 3532)' events.json
```

Output only specific values:

```bash
jq '.[] | select(.event.PROCESS_ID == 3532) | .event.HASH' events.json
```

**Accessing one nest deeper**

```bash
jq '.[] | select(.event.PROCESS_ID == 3532) | .event.PARENT.HASH' events.json
```

