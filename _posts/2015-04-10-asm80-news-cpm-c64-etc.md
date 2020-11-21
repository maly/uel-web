---
id: 487
title: 'ASM80 news: CP/M, C64 etc.'
date: 2015-04-10T12:37:35+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=487
permalink: /2015/04/10/asm80-news-cpm-c64-etc/
categories:
  - Uncategorized
---
I have prepared some new functions in [ASM80.com](http://www.asm80.com).

<!--more-->

New version has some minor issues fixed, i.e. expressions like &#8220;A&#8221;-1 are now evaluated correctly, or LD A,(NNNN) for Z80 is fixed. You can now write expressions such as &#8220;A&#8221;*3 (evaluatred as &#8220;AAA&#8221;). Beautifier (aka &#8220;Format source&#8221;) is fixed too, it aligns columns to the nearest tab. ASM pass 1 is multiplied now, so you can make forward  references now.

But the two biggest news are C64 .PRG generator and CP/M emulator.

## C64 .PRG

You can generate .PRG files for C64 emulators directly, just use these directives:

.PRAGMA PRG &#8211; says &#8220;make .PRG instead of .HEX&#8221;  
.ORG $0810 &#8211; or higher  
.ENT $ &#8211; for &#8220;enter here&#8221;

Here is a short video tutorial (use fullscreen and HD for best quality)



## CP/M stuff

ASM80 has now a generic CP/M emulator. It&#8217;s heavilly inspired by [great Stefan Tramm work](http://www.tramm.li/i8080/). I took his BIOS and slightly modified them to work together with my engine. I took a [CP/M 2.2 source code](http://www.cpm.z80.de/source.html). ASM80 compiled it, and it works. I used also the CP/M boot disk with some utilities (PIP, ED, ZSID) from Stefan Tramm. You can now write a code for Z80 or 8080, use **.PRAGMA COM** and generate a .com file. Use **.engine cpm** to test code in CP/M emulator.

Under the emulator, you can find the boot disk as A:, the work disk with the compiled file as B:

Here is a video tutorial again: