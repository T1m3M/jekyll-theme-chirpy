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

<hr>

## Instruction formats

There are 3 types/formats for registers in MIPS:
- R-Type
- I-Type
- J-Type

### R-Type
<div style="color: #fff; font-weight: bold; text-align: center;">
  <table>
    <tr>
      <td style="background: #3498db;">op</td>
      <td style="background: #c0392b;">rs</td>
      <td style="background: #e67e22;">rt</td>
      <td style="background: #9b59b6;">rd</td>
      <td style="background: #34495e;">shamt</td>
      <td style="background: #3498db;">funct</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>6 bits</td>
    </tr>
  </table>
</div>

- op: opcode/operation (equals 0 in R-type)
- rs: source register
- rt: second source register (since t comes after s in alphabetical order)
- rd: destination source
- shamt: shift amount (for shift instructions)
- funct: function (holds the actual functionality of the instruction for R-Type)

Example:


<div style="color: #fff; text-align: center; font-size: 1.5rem;">
  <span style="color: #3498db;">add</span> <span style="color: #9b59b6;">$s0</span>, <span style="color: #c0392b;">$s1</span>, <span style="color: #e67e22;">$s2</span>
</div>

<div style="color: #fff; font-weight: bold; text-align: center; margin-top: 20px;">
  <table>
    <tr>
      <td>re-arrange</td>
      <td style="background: #3498db;">000000</td>
      <td style="background: #c0392b;">$s1</td>
      <td style="background: #e67e22;">$s2</td>
      <td style="background: #9b59b6;">$s0</td>
      <td style="background: #34495e;">00000</td>
      <td style="background: #3498db;">add</td>
    </tr>
      <tr>
        <td>decimal</td>
        <td style="background: #3498db;">000000</td>
        <td style="background: #c0392b;">17</td>
        <td style="background: #e67e22;">18</td>
        <td style="background: #9b59b6;">16</td>
        <td style="background: #34495e;">00000</td>
        <td style="background: #3498db;">32</td>
      </tr>
    <tr>
      <td>binary</td>
      <td style="background: #3498db;">000000</td>
      <td style="background: #c0392b;">10001</td>
      <td style="background: #e67e22;">10010</td>
      <td style="background: #9b59b6;">10000</td>
      <td style="background: #34495e;">00000</td>
      <td style="background: #3498db;">100000</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>size</td>
      <td>6 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>6 bits</td>
    </tr>
  </table>
</div>

Notes:
- add instruction's function value (32) is from the MIPS manual in which all instruction have a unique code
- the registers values are substituted from the [registers table](http://127.0.0.1:4000/posts/MIPS-cheatsheet/#registers) above

we can say that the instruction machine code is 0x02328020 in hex and this is how it's stored in the executable file or in memory when loaded by the operating system for execution!
