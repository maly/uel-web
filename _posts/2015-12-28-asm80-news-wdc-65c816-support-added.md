---
id: 495
title: 'ASM80 news: WDC 65C816 support added'
date: 2015-12-28T20:50:00+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=495
permalink: /2015/12/28/asm80-news-wdc-65c816-support-added/
categories:
  - Microprocessors
  - My projects
---
Hello from ASM80. Recently (a ten minutes ago, to be honest) I have added a support for WDC&#8217;s 65C816, the 6502 sequel with hybrid data width (8/16 bits). It is used in some notable computers, such as Apple ][gs, Nintendo&#8217;s SNES or SuperCPU extension for C64.

Due to its dual data width, I have to added some new directives to specify index / accumulator width. You have to specify it directly with:

<table>
  <tr>
    <td>
      Directive
    </td>
    
    <td>
      Meaning
    </td>
  </tr>
  
  <tr>
    <td>
      .m8
    </td>
    
    <td>
      Accumulator is 8bit
    </td>
  </tr>
  
  <tr>
    <td>
      .m16
    </td>
    
    <td>
      16bit accumulator
    </td>
  </tr>
  
  <tr>
    <td>
      .x8
    </td>
    
    <td>
      index register is 8bit
    </td>
  </tr>
  
  <tr>
    <td>
      .x16
    </td>
    
    <td>
      16bit index
    </td>
  </tr>
</table>

65C816 added new addresing modes. The main innovation are &#8220;long&#8221; modes with 24bit addresses, e.g. JMP $123456. Such instructions are compiled in 4 bytes, e.g.  &#8220;5C 56 34 12&#8221;.

I believe (the word &#8220;believe&#8221; is important) it should work without any problems, but I warmly appreciate any feedback.

The 65C816 assembler is early beta, so I decided to release it as a single page assembler as this moment instead of full support in IDE. You can try it here: [ASM65816 single page assembler](https://www.asm80.com/onepage/asm65816.html). Please do not hesitate to send any feedback or bugreports to my mail asm80@maly.cz Thanks.