---
title: Marathon CTF
date: 2021-04-02 10:30:00 +0200
categories: [CTF, Writeups]
tags: ctf writeups reverse reverse-engineering web assembly idapro idapython anti-debugging
image: /assets/img/posts/2021-04-02-Marathon-CTF/cover.jpg
---
{% assign img_root = "/assets/img/posts/2021-04-02-Marathon-CTF" %}

Marathon CTF was a great CTF organized by [CyberTalents](https://cybertalents.com/) during the whole month (1 Mar. - 31 Mar. 2021) but I participated during only the last 4 days and I got the 28th place (same points ad 25th) as I solved all Reverse challenges but one, and some of Web, Crypto, Forensics challenges and Here's the writeup for some of these challenges!

<hr>

## Python Art

<img src="{{img_root}}/info/python-art.png" alt="Python Art Info">

Well, This challenge I really liked because It's more like a reverse challenge or at least for me! :D
<img src="{{img_root}}/challenges/python-art/python-art.png" alt="Python Art website">

So this website basically draws "whatever" I typed like this:
<img src="{{img_root}}/challenges/python-art/test.png" alt="testing input">

Since the challenge name is "Python Art" I thought of Django or Flask, I wanted to see the HTML source using inspect element and there was an interesting comment inside the <pre> tag under the drawn text the comment was `<!-- test is not defined -->` or something like that so I guessed the comment is a result of eval()! I tired 2+3 as input and the result in the comment was 5 so YUP, that's eval() function messing with me I KNEW IT!

Because I'm a reverse guy I overthought it so I tried to get the flag variable but it was not defined, I thought it might be in a text file and tried to read `flag.txt` but didn't work either, then I tried to execute compile() function inside the eval function but none of that worked but that made me read about code objects in Python so I had fun.

since the URL is `http://18.196.160.18:3333/send` I tried the input `send` and the comment had this response `<!-- function send at 0xfff..... -->` so that's the address of the function in memory, huh!

so to get the properties and methods of the function send I tried `dir(send)` and the response was
```python
['__call__', '__class__', '__closure__', '__code__', '__defaults__', '__delattr__', '__dict__', '__doc__', '__format__', '__get__', '__getattribute__', '__globals__', '__hash__', '__init__', '__module__', '__name__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', 'func_closure', 'func_code', 'func_defaults', 'func_dict', 'func_doc', 'func_globals', 'func_name']
```

And by taking a look at func_globals by typing `send.func_globals` .. BOOM!:
```
'DEFAULT_FONT': 'standard',
'art': <function art at 0x7f2ce3b7b4d0>,
'Flask': <class 'flask.app.Flask'>,
'app': <Flask 'app'>,
'aprint': <function aprint at 0x7f2ce3b7b450>,
'DECORATION_COUNTER': 195,
'randart': <function randart at 0x7f2ce3b7b550>,
...
lots of gibberish stuff
...
'__file__': 'app.py',
'art_param': <module 'art.art_param' from '/usr/local/lib/python2.7/site-packages/art/art_param.pyc'>
...
'decor_dic': <module 'art.decor_dic' from '/usr/local/lib/python2.7/site-packages/art/decor_dic.pyc'>, 'request': <Request 'http://18.196.160.18:3333/send' [POST]>, 'ART_VERSION': '5.1', 'ART_COUNTER': 700
```

So this is a `Flask` web application written in `app.py` file and there's functions like `art`, `aprint`, `randart`, etc.. which belong to the [art](https://pypi.org/project/art/) python library! All makes sense now! :D

Since I have the filename of source code I grabbed it `open('app.py','r').read()` and styled it a little:

```python
#!/usr/bin/python
from flask import Flask
from flask import render_template
from flask import request
from art import *

app = Flask(__name__, static_url_path='/static')

@app.route('/')
def index():
	return render_template('index.html')


@app.route('/send', methods = ['POST'])
def send():
	text = request.form.get('name')
	art = text2art(str(text),font='block',chr_ignore=True)

	try:
		cmd = eval(text)
	except NameError as e:
	    return render_template('index.html', cmd=e , art= art)
	return render_template('index.html', cmd=cmd , art=art)


if __name__ == '__main__':
	app.run(host='0.0.0.0')
```

so as I thought the comment is using eval() function and all looks good, Now it's time to get the flag!

I typed `__import__('glob').glob("*")` to discover what's around (which I should've done at first BTW xD):
```html
<!-- [&#39;app.py&#39;, &#39;flag&#39;, &#39;templates&#39;, &#39;static&#39;] -->
```
Okay it's `flag` file, not `flag.txt`! then by reading that file `open('flag', 'r').read()` I've got the flag:
<img src="{{img_root}}/challenges/python-art/flag.png" alt="flag">

<hr>

## ASM (4SM)

<img src="{{img_root}}/info/4sm.png" alt="4SM info">

I do love assembly! taking a look at 4SM.asm:
```java
flag_checker:
        push    rbp
        mov     rbp, rsp
        mov     DWORD PTR [rbp-4], edi
        xor     DWORD PTR [rbp-4], 133337
        sar     DWORD PTR [rbp-4], 3
        add     DWORD PTR [rbp-4], 1337
        sub     DWORD PTR [rbp-4], 137
        mov     edx, DWORD PTR [rbp-4]
        mov     eax, edx
        add     eax, eax
        add     eax, edx
        mov     DWORD PTR [rbp-4], eax
        cmp     DWORD PTR [rbp-4], 1128648
        jne     .L2
        mov     eax, 1
        jmp     .L3
.L2:
        mov     eax, 0
.L3:
        pop     rbp
        ret
.LC0:
        .string "Enter the secret number: "
.LC1:
        .string "%d"
.LC2:
        .string "Correct number :D"
.LC3:
        .string "Wrong number :p"
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 16
        mov     edi, OFFSET FLAT:.LC0
        mov     eax, 0
        call    printf
        mov     eax, DWORD PTR [rbp-4]
        mov     esi, eax
        mov     edi, OFFSET FLAT:.LC1
        mov     eax, 0
        call    __isoc99_scanf
        mov     eax, DWORD PTR [rbp-4]
        mov     edi, eax
        call    flag_checker
        mov     DWORD PTR [rbp-8], eax
        cmp     DWORD PTR [rbp-8], 0
        je      .L5
        mov     edi, OFFSET FLAT:.LC2
        call    puts
        jmp     .L6
.L5:
        mov     edi, OFFSET FLAT:.LC3
        call    puts
.L6:
        mov     eax, 0
        leave
        ret
```

The flow goes as follows:
- starting from main
- "Enter the secret number: " message get printed using printf function
- getting user's input through scanf function with %d format
- flag_checker function is called "to check the flag obviously"
- doing some math: `((((x ^ 133337) >> 3) + 1337) - 137) * 3` where x is the user's input
- comparing so the result should be equal to 1128648

_NOTE: the `^` is XOR and `>>` is shift arithmetic right_

I had the equation `((((x ^ 133337) >> 3) + 1337) - 137) * 3 = 1128648` so by solving it:<br>
`x = ((((1128648/3) + 137) - 1337) << 3) ^ 133337 = 3133337`

The flag: `flag{3133337}`

<hr>

## Jett

<img src="{{img_root}}/info/jett.png" alt="Jett info">

I was given an assembly source like the previous challenge:
```java
.LC0:
        .string "Enter the flag:"
.LC1:
        .string "%25s"
.LC2:
        .string "Wrong Flag :("
.LC3:
        .string "Correct Flag : %s"
main:
        push    rbp
        mov     rbp, rsp
        sub     rsp, 160
        mov     DWORD PTR [rbp-160], 150
        mov     DWORD PTR [rbp-156], 204
        mov     DWORD PTR [rbp-152], 233
        mov     DWORD PTR [rbp-148], 31
        mov     DWORD PTR [rbp-144], 211
        mov     DWORD PTR [rbp-140], 75
        mov     DWORD PTR [rbp-136], 212
        mov     DWORD PTR [rbp-132], 74
        mov     DWORD PTR [rbp-128], 193
        mov     DWORD PTR [rbp-124], 215
        mov     DWORD PTR [rbp-120], 85
        mov     DWORD PTR [rbp-116], 176
        mov     DWORD PTR [rbp-112], 132
        mov     DWORD PTR [rbp-108], 215
        mov     DWORD PTR [rbp-104], 57
        mov     DWORD PTR [rbp-100], 222
        mov     DWORD PTR [rbp-96], 38
        mov     DWORD PTR [rbp-92], 75
        mov     DWORD PTR [rbp-88], 2
        mov     DWORD PTR [rbp-84], 139
        mov     DWORD PTR [rbp-80], 75
        mov     DWORD PTR [rbp-76], 21
        mov     DWORD PTR [rbp-72], 215
        mov     DWORD PTR [rbp-68], 21
        mov     DWORD PTR [rbp-64], 229
        mov     edi, OFFSET FLAT:.LC0
        call    puts
        lea     rax, [rbp-48]
        mov     rsi, rax
        mov     edi, OFFSET FLAT:.LC1
        mov     eax, 0
        call    __isoc99_scanf
        mov     DWORD PTR [rbp-4], 0
        jmp     .L2
.L5:
        mov     eax, DWORD PTR [rbp-4]
        cdqe
        movzx   eax, BYTE PTR [rbp-48+rax]
        movsx   eax, al
        imul    eax, eax, 137
        cdq
        shr     edx, 24
        add     eax, edx
        movzx   eax, al
        sub     eax, edx
        mov     DWORD PTR [rbp-8], eax
        mov     eax, DWORD PTR [rbp-4]
        cdqe
        mov     eax, DWORD PTR [rbp-160+rax*4]
        cmp     DWORD PTR [rbp-8], eax
        je      .L3
        mov     edi, OFFSET FLAT:.LC2
        call    puts
        mov     eax, 0
        jmp     .L6
.L3:
        add     DWORD PTR [rbp-4], 1
.L2:
        cmp     DWORD PTR [rbp-4], 24
        jle     .L5
        lea     rax, [rbp-48]
        mov     rsi, rax
        mov     edi, OFFSET FLAT:.LC3
        mov     eax, 0
        call    printf
        mov     eax, 0
.L6:
        leave
        ret
```

The execution flow:
- starting from main
- allocating the stack and storing some bytes
- printing "Enter the flag:" message to the screen using puts
- reading user's input to the address of [rbp-48] using scanf with %25s format
- jumping to .L2 label
- while loop executing while counter [rbp-4] < 25:
  - reading each byte from the user's input (let's call it `x`)
  - doing the math: `y[i] = (byte)[((x[i] * 137) >> 24) + (x[i] * 137)]`
  - known_values[i] = y[i] - ((x[i] * 137) >> 24)
  - increment counter

The full equation is `known_bytes[i] = (byte)[(x[i] * 137) >> 24) + (x[i] * 137)] - ((x[i] * 137) >> 24)` and since ``(x[i] * 137 >> 24)`` for printable characters is always `zero` it can be reduced to: `known_bytes[i] = (byte)(x[i] * 137) = (x[i] * 137) % 256`
and I could brute force them and that's one solution

The other solution is simply using modular inverse to get the flag:
```python
#!/bin/env python3

from Crypto.Util.number import inverse

known_bytes = [150, 204, 233, 31, 211, 75, 212, 74, 193, 215, 85, 176, 132, 215, 57, 222, 38, 75, 2, 139, 75, 21, 215, 21, 229]

flag = ""

for a in known_bytes:
    inv = inverse(137, 256)

    original = inv * a % 256

    flag += chr(original)

print(flag)
```
And I got the flag: `flag{34zy_m0d_1nv3rs3-_-}`

<hr>

## HiKi

<img src="{{img_root}}/info/hiki.png" alt="HiKi info">

I was given an apk file so I decompiled it to jar to see the source code using `dex2jar` tool and there were two interesting source files MainActivity.class and EncDec.class:

<img src="{{img_root}}/challenges/hiki/hiki-main.png" alt="Hiki MainActivity.class">
<img src="{{img_root}}/challenges/hiki/hiki-encdec.png" alt="Hiki EncDec.class">

By reading the code it's obvious that it's doing AES Decryption to a pre-encrypted value and the key is the pin entered by the user concatenated with `Cyb3rT4l3nt5`

Here's the trick the pin in code should be 6 digits or less and since `Cyb3rT4l3nt5` length is 12 and since we're talking about a small key then It's 128-bit key which is 16 bytes we know 12 bytes so the pin should be 4 digits! (so the key is on the format `xxxxCyb3rT4l3nt5`)

The pin has 10k possibilities I wrote a python script to brute force it and simulate the decryption process:

```python
#!/bin/env python3

import base64
from Crypto.Cipher import AES

known_key = "Cyb3rT4l3nt5"

enc = "blJ9s0JObmk7Qi7vfe98RWN3snqSc79GZTC+8IaHuV052R3Q43SH8vXhjQt7E5m3"
b64_enc = base64.b64decode(enc)

for pin in range(0, 10000):

    try:
        key = str(pin).zfill(4) + known_key

        cipher = AES.new(key.encode('utf-8'), AES.MODE_ECB)
        msg = cipher.decrypt(b64_enc).decode('utf-8')

        print("key = {}, flag = {}".format(key, msg))

    except:
        continue
```

<img src="{{img_root}}/challenges/hiki/flag.png" alt="HiKi solver.py">

<hr>

## C1ph3r

<img src="{{img_root}}/info/c1ph3r.png" alt="C1ph3r info">

Last but not least! This is the most challenge I enjoyed so far in the Marathon CTF and you will figure out why!

I was given this 32-bit binary and by analyzing it in IDA and taking a look at the strings:

<img src="{{img_root}}/challenges/c1ph3r/strings.png" alt="strings">

There are a lot of messages that shows the binary is corrupted and need to be fixed also in the API imports `IsDebuggerPresent` function from KERNEL32 Library so I'm expecting ant-debugging techniques!

I went to `"Enter the flag:\n"` reference:

<img src="{{img_root}}/challenges/c1ph3r/fake_flag.png" alt="strings">
<img src="{{img_root}}/challenges/c1ph3r/xor_44.png" alt="fake flag function">

After of pushing data to the stack it calls some functions one of them XORs these bytes with 0x44 _(I named it XOR_44)_ which result in the string `just for your attention, nothing here is real` and yeah just another rabbit hole :"D

Another note from the code overview is that to approach a function it calls a function that jumps to the desired function

I headed back to the beginning to the main function and started tracing statically the flow which was full of exceptions and debugger traps `int 3` instructions to make breakpoints and another important note is *Pseudocode* is not working so we're talking assembly here _**which is cool tbh ;)**_

My approach here was:
1. hitting a breakpoint at the start of try..catch exception
2. modifying EIP to the call instruction (to get out of there ASAP :D)
3. repeating 1 and 2 until reaching finding the treasure (a cool function)

Let's dive in starting from main:

<img src="{{img_root}}/challenges/c1ph3r/main.png" alt="main function">
<img src="{{img_root}}/challenges/c1ph3r/main2.png" alt="main function again">

There were 2 intermediate functions and then I reached this function which seems to be cool the exception start was on the right block but I changed the flow to the highest block above where the magic begins:

<img src="{{img_root}}/challenges/c1ph3r/treasure.png" alt="treasure function">

Here's a closer look to the upper part the first part basically prints "Enter the flag:" and accepts user's input (32 characters) and I could see clearly 2 loops in there one inside another:

<img src="{{img_root}}/challenges/c1ph3r/magic.png" alt="start of magic function">


the first loop iterates while count_1 < 32 (count increases by two) and allocates memory using the standard memset function with size 0x1B28 let's call that `buff` and the second loop iterates while count_2 < 0xD94 (which is half of 0x1B28):

<img src="{{img_root}}/challenges/c1ph3r/loops.png" alt="the two loops">

The second loop is doing encryption and after it finishes it's using memcmp standard function to compare it to a pre-defined memory chunk

<img src="{{img_root}}/challenges/c1ph3r/enc.png" alt="encrypt and compare">

Putting pieces together the first loop takes two characters from the user's input at a time, decrypt it then comparing the result with the pre-stored chunks of memory so I dumped those 16 chunks using 3 lines of IDApython:
```python
start = 0x00CACB60

for counter in range(16):
	idc.savefile("mem" + str(counter).zfill(2) + ".bin", 0, start + (counter * 0x1B28), 0x1B28)
```

Now the decryption process:
- val = two bytes of the user's input
- loop from zero to 0xD94:
  - if val % all_bytes[i] != 0 increment counter
  - otherwise buf[i] += 1 and val = val / all_bytes[i]

Where all_bytes are stored constants in memory to see if these constants divides the value of 2 bytes

So again I made a 2 lines IDApython script to get the all_bytes from memory:
```python
all_bytes_addr = 0x00CAB038

idc.savefile("all_bytes.bin", 0, all_bytes_addr, 0x1B28)
```

Okay enough with the analysis let's talk code this is the encryption simulation in python for first 2 letters of the flag `"fl" = 0x666c` which its result was equal to mem00.bin _(the first chunk of memory)_:
```python
# encrpytion simulation
buf = [ 0 for _ in range(0xD94)]

val = 0x666C
count_2 = 0

while count_2 < 0xD94:

    this_byte = all_bytes[count_2]

    if val % this_byte != 0:
        count_2 += 1;

    else:
        buf[2 * count_2] += 1
        val = int(val / this_byte)

for i in buf:
    print(i, ' ', end='')
```

**Output: 2  0  1  0  1  0  0  0  0  0  0  0  0  0  1  0  1 ... 0** _(which is identical to mem00.bin which holds the first 2 bytes' factorization of the flag)_

Let's sum up, I had the constants and how many times each constant divides the 2 bytes value from the flag so to get those bytes I should calculate the Greatest Common Divisor (GCD) for the 16 values (32 bytes flag) and this way I should have the flag!! so I made a python script to do that for me:
```python
#!/bin/env python3

import struct
from Crypto.Util.number import long_to_bytes

all_bytes = []

# reading all_bytes from the memory dump to array as unsigned short values in little endian
with open("all_bytes.bin", "rb") as f:

    while True:
        buf = f.read(2)

        if not buf:
            break

        all_bytes.append(struct.unpack('<H', buf)[0])



flag_bytes = []

# looping through the 16 memory dumps for the encrypted 2 letters flag
for count_1 in range(16):

    # reading memory dump
    with open("mem/mem" + str(count_1).zfill(2) + ".bin", "rb") as f:
        memdump = f.read()

    # reseting counters
    count_2 = 0
    result = 1

    # getting the divisors from the memory dump
    for ptr in range(0, len(memdump)):

        if memdump[ptr] > 0:

            # how many times the divisor divides the value
            times = memdump[ptr]

            #print("@count_2 = %i -> 0x%x" % (count_2, times))

            # multiplying the divisors to get the GCD (greatest common divisor)
            for _ in range(times):
                result *= all_bytes[count_2]

        elif ptr % 2 == 1:
            count_2 += 1

    flag_bytes.append(result)


# decoding the flag
flag = ""
hex_bytes = ''.join([hex(x)[2:] for x in flag_bytes])
flag = long_to_bytes(int(hex_bytes, 16)).decode('utf-8')
print(flag)
```
<img src="{{img_root}}/challenges/c1ph3r/flag.png" alt="C1ph3r solver.py">
