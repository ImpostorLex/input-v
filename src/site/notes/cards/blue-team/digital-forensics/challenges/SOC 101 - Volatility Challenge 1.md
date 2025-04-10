---
{"dg-publish":true,"permalink":"/cards/blue-team/digital-forensics/challenges/soc-101-volatility-challenge-1/"}
---

[[Challenges TimeKeeping\|Challenges TimeKeeping]]

- Source file [here](https://drive.google.com/file/d/1Vl2mT0ZDb6HrKZqSZKCJjRUjdknWUUL_/view)
- password: infected

time of the snapshot: 2024-10-29 19:18:00+00:00


The other command is not working well for me so I decided to use cmdline:

```C
python3 vol.py -f ~/Desktop/volatility_challenge_1/challenge/challenge.vmem windows.cmdline
```

Then the interesting processes are `crss.exe` which should be located at `system32` not `System` as well as the **correct spelling** is `csrss.exe`:
![SOC 101 - Volatility Challenge 1.png](/img/user/cards/blue-team/digital-forensics/images/SOC%20101%20-%20Volatility%20Challenge%201.png)
At the time of capture, there are three established network connection that came from this binary:
```python
python3 vol.py -f ~/Desktop/volatility_challenge_1/challenge/challenge.vmem windows.netstat | grep -E "5676|3076
```

Output:
![SOC 101 - Volatility Challenge 1-1.png|450](/img/user/cards/blue-team/digital-forensics/images/SOC%20101%20-%20Volatility%20Challenge%201-1.png)
To determine it's parent process we can use the previous command and filter using grep:

```python
python3 vol.py -f ~/Desktop/volatility_challenge_1/challenge/challenge.vmem windows.cmdline | grep -E "3076|5676"
```

Output:
![SOC 101 - Volatility Challenge 1-2.png](/img/user/cards/blue-team/digital-forensics/images/SOC%20101%20-%20Volatility%20Challenge%201-2.png)
It revealed that it's parent process is `explorer.exe`:
```python
python3 vol.py -f ~/Desktop/volatility_challenge_1/challenge/challenge.vmem windows.pslist | grep -E "1560"
```

Output:
![SOC 101 - Volatility Challenge 1-3.png](/img/user/cards/blue-team/digital-forensics/images/SOC%20101%20-%20Volatility%20Challenge%201-3.png)



**Organize and refine notes after each session including adding meta-tags**

