---
{"dg-publish":true,"permalink":"/cards/windows/hook/"}
---

~ [[cards/windows/Windows Internals\|Windows Internals]]
~ [[cards/red-team/Abusing Windows Internals#Memory Execution Alternatives\|Abusing Windows Internals]]

A **hook** is a technique used to **intercept function calls, messages, or events** before they reach their intended destination. In other words a checkpoint done usually by security controls such as Endpoint Detection and Response tools (EDR), **antivirus software**, or **debuggers** to monitor, modify, or block the behavior of a program at runtime.

- Normally: Your program -> `OpenProcess()` -> process opens.
- With a hook:  
    Your program -> **Hooked function** → EDR checks it -> (might allow or deny) ->`OpenProcess()`.

Note: this concept exist in all other OS platforms but imma store them in Windows k.

