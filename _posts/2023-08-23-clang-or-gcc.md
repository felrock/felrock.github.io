---
layout: post
title:  "Dwarfs and Elfs or was it Clang and Gcc"
date:   2023-08-23 12:00:00 +0100
categories: script
---

## Dwarfs and Elfs or was it Clang and Gcc

The other day i was faced with an issue, working with generating A2L files to be used by [XCP](https://en.wikipedia.org/wiki/XCP_(protocol)). Where
XCP is a diagnostics protocol used to read/write to High Integrity machines in the automotive industry.
A2l is a format of the memory to be available for read/write. Its generated from .cpp and .elf files.

In the .cpp files comments are placed on each of the variables that the user would like to read/write to.

Now the issue I was facing is that a fault shown upon generating, which resulted in a seg-fault.

Now, the tool would only work with C-code, however I am using C++ code. Which is a problem, but it
was the only tool around.

I tripple checked, the comments were according to the guidelines of the tool that was used to generate
the a2l files.

The only option I was left with was to debug the actual tool I was using.

So I fired up GDB, and stepped through the tool and found where it was breaking.

I could also mention that the tool started acting up when I added cpp arrays into the mix of generated
signals. So I was pretty sure that was the issue, but I was not at all sure of why the tool broke.

Anyway, it turned out that the tool was expecting a `DW_AT_upper_bound` within the array definition, and
only `DW_AT_count` was available.

`DW_AT` tags are apart of [DWARF](https://dwarfstd.org/), which is a debugging format that reside inside .elf files. I know, I
will tell you about gandalf in a second.

I made a test program to see how arrays look inside the .elf object file.


Here is my test program:
```c
/* main.c */
int main(int argc, char** argv)
{
    volatile int test_ar[500];
    test_ar[0] = 0;
    return test_ar[0];
}
```

Compiling this with __gcc main.c -g -Wall -o main.elf__ and running __objdump -W main.elf__
Using: gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0

In the objdump I can see the array like this:
```elf
 <2><6f>: Abbrev Number: 4 (DW_TAG_variable)
    <70>   DW_AT_name        : (indirect string, offset: 0x12): test_ar
    <74>   DW_AT_decl_file   : 1
    <75>   DW_AT_decl_line   : 3
    <76>   DW_AT_decl_column : 18
    <77>   DW_AT_type        : <0xd0>
    <7b>   DW_AT_location    : 3 byte block: 91 90 70   (DW_OP_fbreg: -2032)
...
 <1><bf>: Abbrev Number: 11 (DW_TAG_array_type)
    <c0>   DW_AT_type        : <0xa7>
    <c4>   DW_AT_sibling     : <0xd0>
 <2><c8>: Abbrev Number: 12 (DW_TAG_subrange_type)
    <c9>   DW_AT_type        : <0xd5>
    <cd>   DW_AT_upper_bound : 499
```

So compiling with gcc results in a `DW_AT_upper_bound` tag as the array size.

But when I compile the same code with clang, __clang main.c -g -Wall -o main.elf__ i get the following elf output.
Using: clang version 10.0.0-4ubuntu1

```elf
 <2><5f>: Abbrev Number: 4 (DW_TAG_variable)
    <60>   DW_AT_location    : 3 byte block: 91 a0 70   (DW_OP_fbreg: -2016)
    <64>   DW_AT_name        : (indirect string, offset: 0x6c): test_ar
    <68>   DW_AT_decl_file   : 1
    <69>   DW_AT_decl_line   : 3
    <6a>   DW_AT_type        : <0xa4>
...
 <1><a4>: Abbrev Number: 8 (DW_TAG_array_type)
    <a5>   DW_AT_type        : <0xb1>
 <2><a9>: Abbrev Number: 9 (DW_TAG_subrange_type)
    <aa>   DW_AT_type        : <0xb6>
    <ae>   DW_AT_count       : 500
```

And there it is, the problem, compilers generate different elf output. Where the array size tag
is `DW_AT_upper_bound` with GCC and `DW_AT_count` with clang.

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
