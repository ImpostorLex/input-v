---
{"dg-publish":true,"permalink":"/cards/windows/syscall/","tags":["windows"]}
---

~ [[map-of-contents/windows\|windows]]

- A "syscall" (System Call) is a programmatic way for a user-space application to request a service from the operating system kernel.

- Operating systems operate in different "privilege levels" (user mode and kernel mode). User-mode programs are restricted for security and stability.

- Syscalls are needed because; to perform privileged operations (printing to the screen, reading a file, exiting the program because only the **kernel** can safely reclaim system resources and update its internal record of running processes), a user-mode program must transition to kernel mode.





