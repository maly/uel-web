---
id: 2269
title: 'ASM80: Debugging stuff'
date: 2017-12-03T13:15:21+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=2269
permalink: /2017/12/03/asm80-debugging-stuff/
image: /wp-content/uploads/2017/12/db0.png
categories:
  - My projects
  - Programming
tags:
  - ASM80
---
I have revisited the [ASM80](https://www.asm80.com/) integrated debugger. I mean that part you can use when try to EMULATE any common code.

The part I have focused on is the upper part with the memory. Here is a change list:

  1. You can click on a memory cell and enter a new value to fill in.
  2. You can scroll memory up and down by mouse wheel.
  3. You can fill a block of memory with three input boxes right under the memory block. From left: FROM, TO, VALUE. Enter appropriate values and click on FILL
  4. You can enter a string or a list of numbers into memory at given addres. The last line provides this functionality. Simply enter the address, from you want to enter into memory, and click to STR (entering ASCII string) or NUMS (entering numbers &#8211; comma-separated list).

Have fun and enjoy!

[  
<img loading="lazy" class="aligncenter size-medium wp-image-2270" src="https://www.uelectronics.info/wp-content/uploads/2017/12/db0-202x300.png" alt="" width="202" height="300" srcset="https://www.uelectronics.info/wp-content/uploads/2017/12/db0-202x300.png 202w, https://www.uelectronics.info/wp-content/uploads/2017/12/db0.png 217w" sizes="(max-width: 202px) 100vw, 202px" />](https://www.uelectronics.info/wp-content/uploads/2017/12/db0.png)