---
title: MIPS cheatsheet
date: 2021-02-22 10:33:00 +0200
categories: [Intro, Assembly]
tags: assembly cheatsheet mips risc low-level
image: /assets/img/posts/2021-02-22-MIPS-cheatsheet/cover.jpg
---
{% assign img_root = "/assets/img/posts/2021-02-22-MIPS-cheatsheet" %}

This is a cheatsheet for MIPS 32-bit, It worth mentioning that MIPS is a RISC (Reduced Instruction Set Computer) architecture with 32 general-purpose registers and 3 instruction formats which you will see in more detail.

MIPS architecture uses 32-bit memory addresses and 32-bit data words (4 bytes), note that the endianness of MIPS can be little or big-endian but we will talk about little-endian here regarding the data represented in memory.

Talking about memory! It's important to know that the addresses of data read or written from/into memory should be word aligned (divisible by 4), now we have a good grasp of MIPS. Let's dive in!

<hr>
