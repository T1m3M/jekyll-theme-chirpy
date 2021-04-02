---
title: Marathon CTF
date: 2021-04-02 00:40:00 +0200
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
