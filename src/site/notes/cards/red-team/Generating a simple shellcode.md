---
{"dg-publish":true,"permalink":"/cards/red-team/generating-a-simple-shellcode/","tags":["red-team"]}
---

~ [[cards/red-team/Evading Antivirus - Shellcode\|Evading Antivirus - Shellcode]]

To generate our own shellcode, we need to write and extract bytes from the assembler machine code.

read: [[cards/windows/syscall\|syscall]]

To write a message in a dialog box, extract bytes from the assembler machine code. and exit the program:

|rax|System Call|rdi|rsi|rdx|
|---|---|---|---|---|
|0x1|sys_write|fd (1=stdout)|pointer to text|length|
|0x3c|sys_exit|exit code|||

- Unlike in any other programming language calling a function/service from the OS in Assembly, we use it's **syscall** number known as **syscall identifiers** instead indicated by the `rax`.

- `rdi`, `rsi`, and `rx` are similar to arguments in Python in this case.
	- To pass the required value of **'pointer to text'** we use the `rsi` register.

Sample

```
mov rax, 0x1        ; sys_write
mov rdi, 1          ; file descriptor for stdout
mov rsi, msg        ; pointer to your string
mov rdx, 13         ; number of bytes to write
syscall             ; do the actual syscall
```

This can be explained with a pseudo code:

> “Hey kernel! I want to run **sys_write** (rax = 1), and here’s my data:  
Write this message (pointer in rsi) to this output (rdi = 1), for this many bytes (rdx = 13).”

Linux syscalls [here](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/).

Compiling first program and obtaining it's shellcode:

![Evading Antivirus.png|450](/img/user/cards/red-team/images/Evading%20Antivirus.png)

a_ label_ in this context is mark the start of a function, reference to data used with jumps and calls.


```C
global _start
section .text
```

- Declares the `_start` label as the **entry point** — this is where the program starts running.
- Everything after `section .text` is **executable code**.

```C
_start:
    jmp MESSAGE
```

- Jump (without condition) to the `MESSAGE` label.
- This skips over `GOBACK:` at first.

```C
MESSAGE:
    call GOBACK
    db "THM, Rocks!", 0dh, 0ah
```

- The `call` pushes the address of the **next instruction** (in this case the memory address of the string) onto the stack.
- Then it jumps to the label `GOBACK`.

```C
GOBACK:
    mov rax, 0x1      ; syscall number for sys_write
    mov rdi, 0x1      ; file descriptor 1 = stdout
    pop rsi           ; get the address of the message (from the stack)
    mov rdx, 0xd      ; 13 bytes
    syscall
```

- `rax = 1`: tell the OS “I want to do `sys_write`”.
- `rdi = 1`: output to stdout (your screen).
    
- `pop rsi`: this pulls the address of the string (remember it was **pushed** meaning at the stop of the stack is the memory address of the string via `call`) 

	- The `pop` takes a value from the top of the stack then stores it into a register or memory location which in this case is `rsi`. This **removes** that address from the stack, **places it into the `rsi` register**.
	- But why?:
	- No need to manually calculate or hardcode a pointer to the string.
	- It's **position-independent** — works even if the binary is relocated in memory.
	- A classic shellcode trick to stay stealthy and compact.

- `rdx = 13` (`0xd`): the number of bytes to print.
    
- `syscall`: make the system call — the string is printed!

```C
    mov rax, 0x3c     ; syscall for exit
    mov rdi, 0x0      ; return code = 0
    syscall
```

- `rax = 60` (0x3c): syscall number for `exit`.
    
- `rdi = 0`: exit with return code 0 (success).
    
- `syscall`: exit the program.

This technique is common in shellcode, where hardcoding addresses is unreliable. Instead, **you find the address of your data dynamically** — like with `call` and `pop`.

Next, we compile and link the ASM code to create an x64 Linux executable file and finally execute the program:

```bash
nasm -f elf64 thm.asm
ld thm.o -o thm
./thm
```

- `nasm` is used to compile the `.asm` file. specifying the `-f elf64` to indicate we are compiling for 64-bits Linux.
- As a result we obtain a `.o` file, the `ld` known as **linker** takes the object file and links it into a full executable and then `-o` output the file as `thm`.

Extracting the shellcode using `objdump` by dumping the `.text` section of the compiled binary:

Yes, everything from `.text` section in our previous code will be extracted:

```C
objdump -d thm
```

Note The above command dumps in memory so there is no file or what, next is extracting the hex value from the above output and store it in `thm.text`:

```C
objcopy -j .text -O binary thm thm.text
```

Finally, our shell code is in binary format, to be able to use it - we need to convert it in hex first using `xdd` command with `-i` switch to output the binary file directly in C string:

```
xxd -i thm.text
```

Output:

![Evading Antivirus-1.png](/img/user/cards/red-team/images/Evading%20Antivirus-1.png)

Our whole code would look like:

```C
#include <stdio.h>

int main(int argc, char **argv) {
    unsigned char message[] = {
        0xeb, 0x1e, 0xb8, 0x01, 0x00, 0x00, 0x00, 0xbf, 0x01, 0x00, 0x00, 0x00,
        0x5e, 0xba, 0x0d, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xb8, 0x3c, 0x00, 0x00,
        0x00, 0xbf, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xe8, 0xdd, 0xff, 0xff,
        0xff, 0x54, 0x48, 0x4d, 0x2c, 0x20, 0x52, 0x6f, 0x63, 0x6b, 0x73, 0x21,
        0x0d, 0x0a
    };
    
    (*(void(*)())message)();
    return 0;
}
```

This is line of code is referred [[cards/red-team/Abusing Windows Internals#Invoking Function Pointers\|here]].
```C
(*(void(*)())message)();
```



