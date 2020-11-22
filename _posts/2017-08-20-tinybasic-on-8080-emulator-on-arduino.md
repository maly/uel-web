---
id: 2254
title: TinyBASIC on 8080 emulator on Arduino
date: 2017-08-20T13:45:28+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=2254
permalink: /2017/08/20/tinybasic-on-8080-emulator-on-arduino/
image: /wp-content/uploads/2017/08/tinybasic2.jpg
categories:
  - AVR
tags:
  - "8080"
  - Arduino
  - TinyBASIC
---
No problem, really.

<!--more-->

  1. Take an Arduino Uno.
  2. Use an emulator from [Altair8800 emulator](https://github.com/companje/Altair8800) (under GPLv2). Not the whole code, just CPU emulation.
  3. Our HAL &#8211; Hardware Abstraction Layer can be really simple. My &#8220;virtual machine&#8221; has a 4kB ROM, mapped in 0x0000-0x0fff (PROGMEM) and an 1 kB RAM, mapped in 0x1000-0x13ff. _You can resize RAM up to cca 1.5 kB)_
  4. Virtual 8080 port 1 is mapped to the real serial port. Virtual port 0xFE is mapped to the built-in LED (just for fun)
  5. Take [a source code](https://www.autometer.de/unix4fun/z80pack/ftp/altair/) for [Tiny BASIC.](https://en.wikipedia.org/wiki/Tiny_BASIC)
  6. Compile it via [ASM80.com &#8211; online assembler](https://www.asm80.com/). It needs two slight modification for MACRO.
  7. Make a binary version as basic.h
  8. Compile, upload, and voila, it works!  
    <a href="https://retrocip.cz/wp-content/uploads/sites/6/2017/08/tinybasic.jpg" rel="lightbox"><img loading="lazy" class="aligncenter size-medium wp-image-967" src="https://retrocip.cz/wp-content/uploads/sites/6/2017/08/tinybasic-650x460.jpg" alt="" width="650" height="460" /></a>
  9. Compile TinyBASIC version 2 too (change port numbers is needed)  
    <a href="https://retrocip.cz/wp-content/uploads/sites/6/2017/08/tinybasic2.jpg" rel="lightbox"><img loading="lazy" class="aligncenter size-medium wp-image-968" src="https://retrocip.cz/wp-content/uploads/sites/6/2017/08/tinybasic2-650x515.jpg" alt="" width="650" height="515" /></a>
 10. Finally: Publish it all [on GitHub](https://github.com/maly/arduino8080basic).

Have a nice day