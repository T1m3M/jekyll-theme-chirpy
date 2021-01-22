---
title: PEscope Tool
date: 2021-01-22 14:54:00 +0200
categories: [Malware Analysis, Tools]
tags: reverse-engineering PE-files malware-analysis tools static-analysis PEscope
image: /assets/img/posts/2021-01-22-PEscope-tool/cover.png
---
{% assign img_root = "/assets/img/posts/2021-01-22-PEscope-tool" %}

This is an official walkthrough with PEscope tool made by me to make basic static analysis more easier, in one place instead of grabbing it to many tools, and surely with cool colors!
The tool is still beta though as i wrote it in less than 2 days!

If you haven't download the tool yet, you can find it here: [PEscope](https://github.com/T1m3M/PEscope)

<hr>

## How it works

The tool can analyze PE files (EXE/DLL) by inspecting the file's PE structure in order to retrieve its information and also the tool provides some hashing and searching for interesting strings such as URLs, emails, IP addresses, etc...<br>
or you can even provide your own regex to search the file's strings!

<hr>

## Full analysis

When providing a PE file to the tool with no arguments it performs full analysis meaning that it will perform all the functionalities the tool can perform as follows:

```console
> pescope foo.exe
```
<div align="center"><img src="{{img_root}}/full-analysis.png" alt="PEscope full analysis"></div>

<hr>

## No colors

If you prefer not to see colors or your OS doesn't support that you can use the ```-c``` flag to disable colors

```console
> pescope -c foo.exe
```
> NOTE: When using this option with other arguments it will also make them colorless!

<div align="center"><img src="{{img_root}}/full-analysis-no-colors.png" alt="PEscope full analysis with no colors"></div>

<hr>

## File information

The ```-i``` flag will get basic information about the file such as:
- File type (EXE/DLL)
- File size (in KB if small)
- 32 or 64-bit
- Number of sections

and so on:
```console
> pescope -i foo.exe
```

<div align="center"><img src="{{img_root}}/info.png" alt="general information about the file"></div>

<hr>

## API libraries

The tool gets the imports of the file and this done explicitly by printing the libraries' names and the functions used within:

```console
> pescope -I foo.exe
```

<div align="center"><img src="{{img_root}}/imports.png" alt="all the file's imports"></div>

or implicitly by printing only the libraries' names:
```console
> pescope -l foo.exe
```

<div align="center"><img src="{{img_root}}/imports-libs.png" alt="the libraries used"></div>

<hr>

## Hashes

It's basically hashes the file using (md5, sha1, sha256) algorithms so if it's confirmed as suspicious it can then be beneficial in IoCs (Indicators of Compromise)

<div align="center"><img src="{{img_root}}/hashes.png" alt="hashing the file"></div>

<hr>

## Sections

As expected it prints information about the file's sections in a cool table:

```console
> pescope -s foo.exe
```

<div align="center"><img src="{{img_root}}/sections.png" alt="The file's sections"></div>

<hr>

## Interesting strings

Because analyzing sample's strings can be like searching for a needle in a haystack this option can save lots of precious analysis time by only looking for URLs, Emails, IP addresses, Errors, Warnings and suspicious words:

```console
> pescope -S bar.exe
```

<div align="center"><img src="{{img_root}}/strings.png" alt="Interesting strings from the file"></div>

<hr>

## Match RegEx

Obviously, Interesting strings option can sometimes be not what the analyst's looking for so here comes the match option to give the analyst more control on the analysis process!

```console
> pescope -m [a-zA-Z]{10,}[\d]$ foo.exe
```

<div align="center"><img src="{{img_root}}/match.png" alt="Matching a regex"></div>

<hr>

## Make your scope!

Now definitely you don't need to run the command for each feature because the main purpose of this tool is to save time! so you can make your mix all in one command!<br>
Example:

```console
> pescope -H -i -s -l foo.exe
```

<div align="center"><img src="{{img_root}}/all-scope.png" alt="Make your scope!"></div>

<hr>

## Conclusion

This was PEscope tool explanation!<br>
I hope that helped you out and if you want to contribute with a feature that will be awesome!<br>
Happy analysis time!
