---
id: 298
title: Online assembler / IDE for old CPUs
date: 2013-12-21T13:47:38+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=298
permalink: /2013/12/21/online-assembler-ide-for-old-cpus/
categories:
  - My projects
  - Z80
tags:
  - ASM80
---
Ladies, gentlemen, let me kindly introduce &#8211; [ASM80](https://www.asm80.com). (Applause!)

<!--more-->

[ASM80](https://www.asm80.com) is an IDE for writting and debugging code for old CPUs 8080, 8085, Z80 and 6502, and it&#8217;s completely online, so it can run directly in your browser (it need no installation). Of course, _modern_ browser (FF, Chrome, IE9+, new Opera) is required.

You can save source files into workspace (which is stored locally in your browser) and compile them into .hex and .lst format. Assembler used in [ASM80](https://www.asm80.com) provides some higher instructions like INCLUDE or MACRO, so you can divide your source code to libraries and include them. You can store your workspaces online as well as download it in ZIP format.

Important: You have to save your file prior to compile. Type of source code is determined by file extension. &#8220;.a80&#8221; means 8080/8085, &#8220;.z80&#8221; means Z80 and &#8220;.a65&#8221; is for 6502 CPU.

Compiling is nice, but [ASM80](https://www.asm80.com) provides debugger too. You can emulate 8080, Z80 or 6502 and walk step-by-step into your code. And more: [ASM80](https://www.asm80.com) contains JavaScript emulators of some vintage microcomputers, so you can test your program in emulator directly. At this time it can emulate only two really obscure Czech microcomputers ([PMI-80](https://www.old-computers.com/museum/computer.asp?c=1016&st=1) and [PMD-85](https://en.wikipedia.org/wiki/PMD_85)), but I work on further emulators (KIM-1, ZX80, ZX81, ZX Spectrum, C64) .

[ASM80](https://www.asm80.com) is _work-in-progress_ and new functions are added each week. The main goal I&#8217;d like to fulfill is documentation in English (I have only Czech documentation).

So here is my [Retrochallenge](https://www.wickensonline.co.uk/retrochallenge-2012sc/): Write a documentation and prepare some new emulators!

Happy retrocoding!