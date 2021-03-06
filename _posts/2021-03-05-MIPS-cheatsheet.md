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

Talking about memory! It's important to know that the addresses of data read or written from/into memory should be word aligned (divisible by 4), now we have a good grasp of MIPS.

Before we dive in. It worth mentioning that most of the examples in this post is from [Digital Design and Computer Architecture 2nd edition by David Harris and Sarah Harris](https://www.amazon.com/Digital-Design-Computer-Architecture-Harris-ebook/dp/B00HEHG7W2) which I highly recommend for those interested in computer architecture.

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

Examples:

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
    <tr>
      <td colspan="7">0x02328020</td>
    </tr>
  </table>
</div>

<div style="color: #fff; text-align: center; font-size: 1.5rem;">
  <span style="color: #3498db;">sll</span> <span style="color: #9b59b6;">$s0</span>, <span style="color: #e67e22;">$s2</span>, <span style="color: #34495e;">2</span>
</div>

<div style="color: #fff; font-weight: bold; text-align: center; margin-top: 20px;">
  <table>
    <tr>
      <td style="background: #3498db;">000000</td>
      <td style="background: #c0392b;">00000</td>
      <td style="background: #e67e22;">18</td>
      <td style="background: #9b59b6;">16</td>
      <td style="background: #34495e;">2</td>
      <td style="background: #3498db;">000000</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>6 bits</td>
    </tr>
    <tr>
      <td colspan="6">0x02328020</td>
    </tr>
  </table>
</div>

Notes:
- add instruction's function value (32) and (0) for sll is from the MIPS manual in which all instruction have a unique code
- the registers values are substituted from the [registers table](#registers) above

we can say that the instruction machine code is 0x02328020 in hex and this is how it's stored in the executable file or in memory when loaded by the operating system for execution!

### I-Type

<div style="color: #fff; font-weight: bold; text-align: center;">
  <table>
    <tr>
      <td style="background: #3498db;">op</td>
      <td style="background: #c0392b;">rs</td>
      <td style="background: #e67e22;">rt</td>
      <td style="background: #27ae60;">immediate</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>16 bits</td>
    </tr>
  </table>
</div>

Examples:

<div style="color: #fff; text-align: center; font-size: 1.5rem;">
  <span style="color: #3498db;">addi</span> <span style="color: #e67e22;">$s0</span>, <span style="color: #c0392b;">$s1</span>, <span style="color: #27ae60;">5</span>
</div>

<div style="color: #fff; font-weight: bold; text-align: center; margin-top: 20px;">
  <table>
    <tr>
      <td style="background: #3498db;">8</td>
      <td style="background: #c0392b;">17</td>
      <td style="background: #e67e22;">16</td>
      <td style="background: #27ae60;">5</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>16 bits</td>
    </tr>
    <tr>
      <td colspan="4">0x22300005</td>
    </tr>
  </table>
</div>


<div style="color: #fff; text-align: center; font-size: 1.5rem;">
  <span style="color: #3498db;">sw</span> <span style="color: #e67e22;">$s1</span>, <span style="color: #27ae60;">4</span>(<span style="color: #c0392b;">$t1</span>)
</div>

<div style="color: #fff; font-weight: bold; text-align: center; margin-top: 20px;">
  <table>
    <tr>
      <td style="background: #3498db;">43</td>
      <td style="background: #c0392b;">9</td>
      <td style="background: #e67e22;">17</td>
      <td style="background: #27ae60;">4</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>5 bits</td>
      <td>5 bits</td>
      <td>16 bits</td>
    </tr>
    <tr>
      <td colspan="4">0xAD310004</td>
    </tr>
  </table>
</div>

### J-Type

a special type for J (Jump) and JAL (Jump and link) instructions.

`NOTE: (Jump Register) JR instruction is an R-Type instruction with only rs operand assigned`

<div style="color: #fff; font-weight: bold; text-align: center;">
  <table>
    <tr>
      <td style="background: #3498db;">op</td>
      <td style="background: #f1c40f;">address</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>26 bits</td>
    </tr>
  </table>
</div>

Example:

<div style="color: #fff; text-align: center; font-size: 1.5rem;">
  <span style="color: #3498db;">jal</span> <span style="color: #f1c40f;">loop</span>
</div>

<div style="color: #fff; font-weight: bold; text-align: center; margin-top: 20px;">
  <table>
    <tr>
      <td style="background: #3498db;">3</td>
      <td style="background: #f1c40f;">0x100028</td>
    </tr>
    <tr style="background: none; border: none;">
      <td>6 bits</td>
      <td>26 bits</td>
    </tr>
    <tr>
      <td colspan="2">0x0C100028</td>
    </tr>
  </table>
</div>

Note that the label address is represented in pseudo-direct addressing to make it possible to write a 32-bit address in only 26-bits which will be discussed right now!

<hr>

## Addressing modes

There are 5 different ways the CPU can access the memory in MIPS

### Register-only
registers for all source and destination operands (R-Type uses it)

### Immediate addressing
16-bit immediate with registers as operands (i.e. addi and lui)

### Base-addressing
memory access instructions (i.e. lw and sw)

address of memory = base + sign-extended 16-bit offset of immediate

Example: lw $s0, 8($s1) `address = $s1 (base-pointer) + 8`

### PC-relative
conditional branch instructions (i.e. beq, bne, ...) use it to compute the new value of the PC _(Program Counter)_

`Branch Target Address (BTA) = (PC + 4) + sign-extended offset of immediate`

so if the offset is negative the label is above the current instruction.

Example:
```
0xA4 beq $t0, $0, else
0xA8 addi $v0, $0, 1
0xAC addi $sp, $sp, 8
0xBO jr $ra
0xB4 else: addi $a0, $a0, −1
0xB8 jal factorial
```

assuming PC=0xA4 the BTA will be: `(0xA4 + 4) + 3 instructions` which means the target address is 3 instructions after 0xA8 instruction (if it's a -5 then it would be 5 instructions _before_ 0xA4)

### Pseudo-direct
here the address is specified in the instruction which is used in J and JAL instructions (J-Type instructions) recall the example in [J-Type](#j-type) the address had only 26-bits to be stored while in a program it should be 32-bit address for PC!

That's the algorithm to calculate a 26-bit address from 32-bit address:

1- get the address of the label instruction _Jump Target Address (JTA)_

2- Discard the 2 least significant bits JTA<sub>1:0</sub> **Because the instructions are word-aligned 4 (0100)<sub>2</sub>, 8 (1000)<sub>2</sub>, 12 (1100)<sub>2</sub> so the 2 LSB are always zeros!)**

3- Discard the 4 most significant bits JTA<sub>31:28</sub> **Because they can be obtained from the PC address so if your program is not long it won't be far from current instruction which also puts some constraints on the range)**

Example:

```
0x0040005C jal sum
...
0x004000A0 sum: add $v0, $a0, $a1
```

The JTA for JAL instruction here is 0x004000A0 and here's the conversion of it to 26-bit address by applying the above algorithm:

<div style="color: #fff; font-weight: bold; text-align: center; margin-top: 20px;">
  <table>
    <tr>
      <td style="background: #a55eea;">0</td>
      <td style="background: #8854d0;">0</td>
      <td style="background: #a55eea;">4</td>
      <td style="background: #8854d0;">0</td>
      <td style="background: #a55eea;">0</td>
      <td style="background: #8854d0;">0</td>
      <td style="background: #a55eea;">A</td>
      <td style="background: #8854d0;">0</td>
    </tr>
    <tr>
      <td style="background: #a55eea; text-decoration: line-through;">00 00</td>
      <td style="background: #8854d0;">00 00</td>
      <td style="background: #a55eea;">01 00</td>
      <td style="background: #8854d0;">00 00</td>
      <td style="background: #a55eea;">00 00</td>
      <td style="background: #8854d0;">00 00</td>
      <td style="background: #a55eea;">10 10</td>
      <td style="background: #8854d0;">00<span style="text-decoration: line-through;">00</span></td>
    </tr>
    <tr>
      <td colspan="8">0x0100028</td>
    </tr>
  </table>
</div>

Note that the address is combined into hex from right to left. `You can think of the reverse process of adding 00 to the most right and adding the 4 most significant bits from the PC which are zero to get the original address 0x004000A0 back!`

<hr>

## Variables and Arrays

### Variables
To store a variable you can store it "immediately" (I-Type) to a registers if it's 16 bits or less

```
addi $s0, $0, $0xF00D # $s0 = 0xF00D
```

or if it's 32 bits:
```
lui $s0, 0x1337 # $s0 = 0x13370000
ori $s0, $s0, 0xF00D # $s0 |= 0xF00D = 0x1337F00D
```

or you can store it in memory in the data section and load it into a register:
```
.data
  num: .word 0x1337F00D

.text
.globl main

main:
  la $t0, num # $t0 = &num
  lw $s0, ($t0) # $s0 = num = 0x1337F00D
```

note that .word in the data section is the size of the variable (word = 4 bytes = 32 bits) and can be:
- .space (empty)
- .byte (8 bits)
- .word (4 bytes)
- .asciiz (null terminated string)
- .ascii (string without null terminator)
- .align (aligns the next data on a 2<sup>n</sup> byte boundary)

### Arrays

The array is stored in memory in the data section:

```
.data
  arr: .word 1, 2, 3

.text
.globl main

main:
  la $s0, arr 	 # $s0 = base address of arr
  lw $t0, 0($s0) # $t0 = arr[0] = 1
  lw $t1, 4($s0) # $t0 = arr[1] = 2
  lw $t2, 8($s0) # $t0 = arr[2] = 3

exit:
  li $v0, 10
  syscall
```

note that the string is nothing but an array of characters in memory!

<hr>

## Multiplication and division

Since multiplication of two 32-bit registers can result in a 64-bit value, and the division of two numbers results in a quotient and a remainder there are 2 special purpose registers for mult and div instructions:
```
mult $s0, $s1 # result = hi:lo
div  $s0, $s1 # hi = quotient, lo = remainder
```

<hr>

## If / else conditions

The main observation here is `the low level condition is the inverse of the high level condition`

High-level code:
```
if (x == y):
  x = y << 2
else:
  x = y
```

Low-level code

```
..
  addi $s0, $0, 0     #x = 0
  addi $s1, $0, 1     #y = 1
  bne  $s0, $s1, else #if (x != y) go to else
  sll  $s0, $s1, 2    #x = (x << 2)
  j    done

else:
  add $s0, $s0, $s1   #x = y

done:
..
```

Note that this code contains a conditional jump `bne` for comparison and an unconditional jump `j` to prevent from executing the else statement since assembly goes naturally from top to bottom!

<hr>

## Loops

In programming loops consists of 4 main pieces:
- Initialization
- Condition
- Code that gets repeated
- Counter update

and so do assembly!

High-level code:
```
int pow = 1;         // (initialization)
int x = 0;

while (pow != 128) { // (condition)
  pow = pow * 2;     // (code that gets repeated)
  x = x + 1;         // (counter update)
}
```

Low-level code

```
..
  addi $s0, $0, 1    # pow = 1 (initialization)
  addi $s1, $0, 0    # x = 0
  addi $t0, $0, 128  # t0 = 128 for comparison

while:
  beq $s0, $t0, done # if pow == 128, exit while loop (condition)
  sll $s0, $s0, 1    # pow = pow * 2 (code that gets repeated)
  addi $s1, $s1, 1   # x = x + 1 (counter update)
  j while

done:
..
```

<hr>

## Functions

There's no call/ret instructions in MIPS assembly like in Intel x86, but fortunately we have got labels! with this and Jump and Link `JAL` and Jump Register `JR` we can do magic!

### Function call

Jump and Link `JAL` copies the next instruction address (PC+4) to the `$ra` register and jumps to the address of the label, when returning the Jump Register `JR $ra` copies the value of $ra back to the PC so the program continues execution from where it was left.

This is an implementation for a basic function that adds two numbers and returns the result:

High-level code:
```
int add_func(x, y) {
  return x + y;
}

int main() {
  a = add_func(2, 4);
}
```

Low-level code:
```
.text
.globl main
add_func:
  add $v0, $a0, $a1
  jr $ra            # return (PC = $ra)

main:
  addi $a0, $0, 2
  addi $a1, $0, 4
  jal add_func      # call function ($ra = PC + 4, PC --> add_func)
  add $s0, $0, $v0
```

Notes:
- saved registers ($s0-$s7) shouldn't change after call
- temporary registers ($0-$t9) can be changed inside the function
- arguments should be passed in ($a0-$a3) registers
- return values should be saved in ($v0-$v1) registers

### Stack frames

each function should have its stack frame for purposes like saving the saved registers ($s0-$s7) at first and retrieving the values back at the end, and to control the stack we have 2 registers for this task:

- `$fp`: base of the stack
- `$sp`: top of the stack

Note that `$fp` > `$sp` since **`the stack grows towards lower memory addresses`**

Example of allocating and de-allocating stack frame of a function:

 ```
 addi $sp, $sp, -12 # allocation
 sw   $s0, 8($sp) # saves $s0
 sw   $t0, 4($sp) # saves $t0
 sw   $t1, 0($sp) # saves $t1
 ..
 lw   $t1, 0($sp) # restores $t1
 lw   $t0, 4($sp) # restores $t0
 lw   $s0, 8($sp) # restores $s0
 addi $sp, $sp, 12 # de-allocation
 ```
