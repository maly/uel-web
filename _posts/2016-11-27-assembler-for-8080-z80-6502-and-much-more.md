---
id: 2209
title: 'Assembler for 8080, Z80, 6502 and much more&#8230;'
date: 2016-11-27T10:23:41+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=2209
permalink: /2016/11/27/assembler-for-8080-z80-6502-and-much-more/
suf_magazine_headline:
  - 'on'
categories:
  - "6502"
  - Microprocessors
  - My projects
  - Programming
  - Z80
tags:
  - "6502"
  - "65816"
  - "6800"
  - "6809"
  - "8080"
  - ASM80
  - assembler
  - Z80
---
Readers certainly know my [ASM80](https://www.asm80.com/) &#8211; online assembler / IDE for eight-bit processors. I made several derived versions, like single page compiler, embedded version of the translator (I used it in tutorial [Stroják.cz](https://strojak.cz/)), or [a stand-alone IDE](https://www.ide80.com/). The good old command line assembler is here now too.

The prerequisite you have to meet is a functional Node.js environment. It is not complicated, it exists for all major platforms, and you can download it here: [nodejs.org](https://nodejs.org). During installation, package manager called NPM is installed too.

NPM is used for install the packages and libraries. To install ASM80 itself just run a command prompt and type:

<pre class="">NPM i -g asm80</pre>

-g causes the asm80 will not be installed as a library, but as a system tool. Then toy can invoke it as a standard command line utility:

<pre class="">asm80 test.a80</pre>

launches translation for file test.a80 and the result will be two files: test.hex with output and test.lst with the translation protocol. Extension .a80 tells the compiler to use the processor&#8217;s instruction set for Intel 8080 CPU.

Behaviour can be influenced by parameters. You can set the output file name, you can suppress the generation .lst, or explicitly determine the processor type and format of the output file (besides HEX and S record it can output .COM files for CP/M, .PRG for C64 emulators, or SNA and TAP for ZX Spectrum).

Options are:

  * `-o, --output <file>` Output file name
  * `-t, --type <type>` Output type [default: hex]. Available types are: hex, srec, com (for CP/M), sna, tap (for ZX Spectrum), prg (for C64)
  * `-n, --nolist` Suppress listing (.lst file)
  * `-m, --machine <type>` Processor type, one of the following: Z80, I8080, C6502, C65816, CDP1802, M6800, M6809
  * `-h, --help` See HELP

Machine type can be omitted. Right CPU is determined by file name extension (-m option overrides this decision).

  * Intel 8080: .A80
  * Zilog Z80: .Z80
  * Motorola 6800: .A68
  * Motorola 6809: .A09
  * MOS 6502: .A65
  * WDT 65816: .816
  * CDP 1802: .A18

These parameters are described on [page NPM package ASM80](https://www.npmjs.com/package/asm80).

Overview of the syntax and directives can be found on [GitHub Pages](https://maly.github.io/asm80-node/).

I still have a few suggestions for improvements, I would like to know your opinion &#8230;

  * Create a library system, as it has the classic assemblers, which separates translation and linking. So users will have the opportunity to make a library of subroutines, which would include only those parts of the code that are necessary for proper function. You can easily make something like the &#8220;standard C library&#8221; for your system &#8230;
  * Having the opportunity to directly link public code, for example on GitHub.
  * &#8230; More processors? Systems?

Thanks for the tips and suggestions. You can send them straight to the [GitHub Issues](https://github.com/maly/asm80-node/issues).