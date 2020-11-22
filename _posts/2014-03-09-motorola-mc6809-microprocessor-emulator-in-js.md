---
id: 302
title: Motorola MC6809 microprocessor emulator in JS
date: 2014-03-09T09:18:38+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=302
permalink: /2014/03/09/motorola-mc6809-microprocessor-emulator-in-js/
categories:
  - Uncategorized
tags:
  - ASM80
---
Here is another part of my [ASM80.com](https://www.uelectronics.info/2013/12/21/online-assembler-ide-for-old-cpus/ "Online assembler / IDE for old CPUs") free licensed: a [6809 emulator in JavaScript](https://github.com/maly/6809js).

<!--more-->

It&#8217;s alpha version, not optimized yet, no interrupts handled and CWAI instruction, but it works. I&#8217;ve tested it on [Grant Searle&#8217;s 6809 simple SBC](https://searle.hostei.com/grant/6809/Simple6809.html). The next step is Hitachi HD6309, so stay tuned!

## Usage

(a.k.a. The API)

  * _window.CPU6809_ &#8211; main object (instantiated at the start &#8211; it shall change)
  * _CPU6809.init(memoryTo,memoryAt,ticker)_ &#8211; Initializes the whole system. All parameters are callback functions for port / memory access: 
      * memoryTo(addr,value) &#8211; store byte to given address
      * memoryAt(addr) &#8211; read byte from given address
      * ticker(T) &#8211; unused now. For future use
  * _CPU6809.T()_ &#8211; returns clock ticks count from last init (or reset)
  * _CPU6809.reset()_ &#8211; does a CPU reset
  * _CPU6809.set(register, value)_ &#8211; sets internal register (named PC, SP, U, A, B, X, Y, DP and flags) to a given value
  * _CPU6809.status()_ &#8211; Returns a object {pc, sp, u, a, b, x, y, dp} with actual state of internal registers
  * _CPU6809.steps(N)_ &#8211; Execute instructions as real CPU, which takes &#8220;no less than N&#8221; clock ticks.