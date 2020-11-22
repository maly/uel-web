---
id: 2264
title: 'ASM80: New directives, some fixes, new login system'
date: 2017-09-16T11:06:43+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=2264
permalink: /2017/09/16/asm80-new-directives-some-fixes-new-login-system/
categories:
  - My projects
  - Programming
tags:
  - ASM80
---
Hello folks, as you can noticed, there is some progress with [ASM80.com](https://www.asm80.com) online IDE and compiler.

Last time I have introduced new features: [generic emulator](https://www.uelectronics.info/2017/09/03/generic-emulators-for-asm80-com/) and [assembler toolkit](https://www.uelectronics.info/2017/09/09/assembler-toolkit/).

Today I would like to introduce some features you will love, I guess&#8230;

First of all: [ASM80 Manual](https://maly.gitbooks.io/asm80/), published with GitBook. Enjoy!

### Directives

I have added three directives: .cstr, .pstr and .istr. All of them is suitable for defining a string, like _DB &#8220;blahblah&#8221;_, but:

<table>
  <tr>
    <td>
      .cstr
    </td>
    
    <td>
      When you need write a zero-ended string (C style), you can use <code>DB "Hello",0</code> &#8211; or simple <code>.cstr "Hello"</code>
    </td>
  </tr>
  
  <tr>
    <td>
      .pstr
    </td>
    
    <td>
      Similar as .cstr, but there is no trailing zero. .pstr is a Pascal-style string: first byte is length, then string. So <code>.pstr "Hello"</code> is equal to <code>DB 5, "Hello"</code>.
    </td>
  </tr>
  
  <tr>
    <td>
      .istr
    </td>
    
    <td>
      Strings are often defined as simple ASCII, where the last byte has bit 7 set to 1. So <code>.istr "Hello"</code> is the same as <code>DB "Hell","o"+0x80</code>
    </td>
  </tr>
</table>

### Terminal capturing and transmitting

With Generic Emulator you can now easy capture terminal out to the text file (press &#8220;capture&#8221; under the display, press again to stop capturing and save file). You can send file to terminal too, just click to the file name with RMB and select &#8220;Send to terminal&#8221;

### New login system

I have decided to deprecate old login system, it was not good. From now you can log in with GitHub account, Twitter account, Google account or Facebook account. You can link more such accounts together, of course. Your workspaces is now saved in Firebase DB. Your old workspaces are preserved, but it requires old way of login, so I recommend to save them into the new system.

### Bug fixed

There was a bug in a .block feature &#8211; in some circumstances with 6502 code assembler can decide to use shorter addressing mode, so .block can be moved during the evaluation passes to another address. Now assembler reflect these changes and provide true addresses for inner @labels.