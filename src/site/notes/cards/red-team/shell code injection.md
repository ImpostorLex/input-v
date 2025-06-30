---
{"dg-publish":true,"permalink":"/cards/red-team/shell-code-injection/","tags":["red-team/concept"]}
---

~ [[cards/red-team/Abusing Windows Internals\|Abusing Windows Internals]]

_Shellcode injection_ is a technique where malicious code (shellcode) is injected into a running process's memory and then executed. This is used to **evade detection**, **run code under another process’s identity**, or **bypass endpoint protection**.

Once the shellcode is injected into a process and executed by the vulnerable software or program, it modifies the code run flow to update registers (Small storage areas on RAM) and functions of the program to execute the attacker's code.

It's generally written in Assembly languages and translated into hexadecimal opcodes, unique and custom shellcode help evading AV software however writing shellcode is not easy.