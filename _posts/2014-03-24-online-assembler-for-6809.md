---
id: 329
title: Online assembler for 6809
date: 2014-03-24T14:38:33+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=329
permalink: /2014/03/24/online-assembler-for-6809/
categories:
  - Microprocessors
  - My projects
tags:
  - "6809"
  - ASM80
  - Motorola
---
I&#8217;ve updated my [online assembler IDE](https://www.asm80.com) recently ([more info here](https://www.uelectronics.info/2013/12/21/online-assembler-ide-for-old-cpus/)). It can assemble / debug code for MCU Motorola 6809. The assembler is in alpha version, not yet fully compatible with old 6809 assemblers (pseudoinstructions like FCB, FCC, BSZ, RMB, ZMB are missing, but I&#8217;m working on them). The all you need is to create a file with &#8220;.a09&#8221; extension and compile it. Compiler outputs to &#8220;HEX&#8221; files (not S records yet, but it&#8217;ll be soon). Assembled code can be tested in embedded 6809 emulator, based on [my code](https://www.uelectronics.info/2014/03/09/motorola-mc6809-microprocessor-emulator-in-js/). You can use an [online SBC09 emulator](https://www.asm80.com/sbc09.html), which is capable to emulate [Grant Searle&#8217;s 6 chip simple 6809 computer](https://searle.hostei.com/grant/6809/Simple6809.html). Stay tuned for further information&#8230;