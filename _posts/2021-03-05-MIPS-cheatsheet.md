---
title: MIPS cheatsheet
date: 2021-03-05 13:58:00 +0200
categories: [Intro, Assembly]
tags: assembly cheatsheet mips risc low-level
image: /assets/img/posts/2021-03-05-MIPS-cheatsheet/cover.jpg
---
{% assign img_root = "/assets/img/posts/2021-03-05-MIPS-cheatsheet" %}

This is a cheatsheet for MIPS 32-bit, It worth mentioning that MIPS is a RISC (Reduced Instruction Set Computer) architecture with 32 general-purpose registers and 3 instruction formats which you will see in more detail.

MIPS architecture uses 32-bit memory addresses and 32-bit data words (4 bytes), note that the endianness of MIPS can be little or big-endian but we will talk about little-endian here regarding the data represented in memory.

Talking about memory! It's important to know that the addresses of data read or written from/into memory should be word aligned (divisible by 4), now we have a good grasp of MIPS. Let's dive in!

<hr>

## Registers

| Name    | Number | Use                               |
| ------- | ------ | --------------------------------- |
| $0      |	0      |	constant 0                       |
| $at     | 1      | assembler temporary               |
| $v0–$v1 | 2–3    | function return value             |
| $a0–$a3 | 4–7    | function arguments                |
| $t0–$t7 | 8–15   | temporary variables               |
| $s0–$s7 | 16–23  | saved variables                   |
| $t8–$t9 | 24–25  | temporary variables               |
| $k0–$k1 | 26–27  | operating system (OS) temporaries |
| $gp     | 28     | global pointer                    |
| $sp     | 29     | stack pointer                     |
| $fp     | 30     | frame pointer                     |
| $ra     | 31     | function return address           |
