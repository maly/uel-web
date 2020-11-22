---
id: 447
title: An introduction to ASM80
date: 2014-11-15T13:09:57+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=447
permalink: /2014/11/15/an-introduction-to-asm80/
image: /wp-content/uploads/2014/11/asm80.png
categories:
  - My projects
  - Programming
tags:
  - ASM80
---
A long time ago I started to play with 8bit CPUs and old computers&#8230; again.

There is a lot of development tools for these computers, a tons of assemblers and utilities for any OS, from CP/M to Mac OS, DOS, Linux and Windows including. So there is no space for a new one. But internet is a magic &#8220;big space for everything&#8221;. 🙂

When I bought [Phil&#8217;s V6Z80P](https://www.retroleum.co.uk/electronics-articles/v6z80p/), I used [PASMO assembler](https://pasmo.speccy.org/). It is a great state-of-the-art assembler, really perfect. But over time I had a lot of assemblers, compilers and IDEs for different CPUs somewhere on my disk. And one day my disk crashed. So I wished to have &#8220;an online PASMO&#8221;.

At first I made online editor with server-side PASMO. You wrote a code in your browser, pressed COMPILE, code went to the server, PASMO did its magic and send back my HEX (or BIN or whatever). Great thing! Later I&#8217;ve added FLASH-based Spectrum emulator, so I could write programs for ZXS, compile it and try it directly in emulator. But I never published this tool.

One year ago I&#8217;ve decided to reinvent my wheel again, and this time without server back support. I choose text editor in JavaScript and start development of my very own assembler engine. My goal was to create a modular assembler for different 8bit CPUs, from 8080 to 6809, with simillar syntax. I&#8217;ve integrated it with the editor into one &#8220;IDE&#8221;. Later I added CPU emulating engines, so you could debug your compiled code. Then I added whole computer emulations&#8230;

And all of this is free, online, partially open source.

The main part is [ASM80.com](https://www.asm80.com). This is a web-based IDE with following features:

  * Code editor
  * Simulated workspace for your files (it&#8217;s stored technically in your browser, so you haven&#8217;t to got any &#8220;server account&#8221;)
  * Compiling engines for 8080/8085, Z80, 6502, 6800 and 6809 CPU
  * Embedded emulators for all these CPUs
  * Emulators for some computers: 
      * Czech computers [PMI-80](https://en.wikipedia.org/wiki/PMI-80), [PMD-85](https://en.wikipedia.org/wiki/PMD_85), JPR-1 (also known as [SAPI-1](https://en.wikipedia.org/wiki/SAPI-1))
      * ZX Spectrum
      * [Grant Searle&#8217;s Single Board Computers](https://searle.hostei.com/grant/) with 6502, 6809 and Z80
  * Nearly the same syntax and directives for all CPUs
  * Macros
  * Preprocessor

Here is [an assembler description](https://www.uelectronics.info/2014/03/30/asm80-description/) and here is [source code for the engine](https://github.com/maly/asm80).

During my work on ASM80 I developed some minor things in JavaScript. All of these are open source too: [6850js](https://github.com/maly/6850js) (6850 emulator), [6809 emulator](https://github.com/maly/6809js) or enhanced [8080js emulator](https://github.com/maly/8080js), which can pass through the Examiner test (so every bit works, every T cycle is valid&#8230;)

Then I&#8217;ve derived [small one-page assemblers](https://www.uelectronics.info/2014/10/07/8080-6502-6809-and-z80-assembler-as-a-html-page/) for all supported CPUs. You can save it in your Favorites or at your local disk and use for simple compiling.

And the newest part of ASM80 family is [an IDE80](https://www.ide80.com/). It&#8217;s technically ASM80, with IDE, assembler, debugger and emulatore, repacked and compiled as a desktop application. It&#8217;s in beta version, just for Windows at this time, but Linux and Mac OS X versions should came very soon. You can watch a video showing first steps with this IDE.



At this time I&#8217;m working on further improvements:

  * Structures in assembler
  * C64 or Atari emulator in JavaScript
  * <del>KIM-1 emulator</del>
  * modules for generating not just HEX or S19, but TAP, SNA, PRG and other popular formats for 8bit computers.

Update 11/16/2014: Beta version of KIM-1 emulator is ready and deployed&#8230;





&nbsp;

<!-- see gallery_shortcode() in wp-includes/media.php -->

<div id='gallery-1' class='gallery galleryid-447 gallery-columns-3 gallery-size-thumbnail gallery1'>
  <dl class="gallery-item">
    <dt class="gallery-icon">
      <a href="https://www.uelectronics.info/wp-content/uploads/2014/11/asm1.png" title="ASM80 with source code" rel="gallery1"><img src="https://www.uelectronics.info/wp-content/uploads/2014/11/asm1-150x150.png" width="150" height="150" alt="ASM80 with source code" /></a>
    </dt>
    
    <dd class="gallery-caption" id="caption453">
      <span class="imagecaption">ASM80 with source code</span><br />
    </dd>
  </dl>
  
  <dl class="gallery-item">
    <dt class="gallery-icon">
      <a href="https://www.uelectronics.info/wp-content/uploads/2014/11/asm2.png" title="Compiling..." rel="gallery1"><img src="https://www.uelectronics.info/wp-content/uploads/2014/11/asm2-150x150.png" width="150" height="150" alt="Compiling..." /></a>
    </dt>
    
    <dd class="gallery-caption" id="caption452">
      <span class="imagecaption">Compiling...</span><br />
    </dd>
  </dl>
  
  <dl class="gallery-item">
    <dt class="gallery-icon">
      <a href="https://www.uelectronics.info/wp-content/uploads/2014/11/asm4.png" title="Grant Searle&#039;s SBC6809 emulator" rel="gallery1"><img src="https://www.uelectronics.info/wp-content/uploads/2014/11/asm4-150x150.png" width="150" height="150" alt="Grant Searle&#039;s SBC6809 emulator" /></a>
    </dt>
    
    <dd class="gallery-caption" id="caption450">
      <span class="imagecaption">Grant Searle's SBC6809 emulator</span><br />
    </dd>
  </dl>
  
  <br style="clear: both" />
  
  <dl class="gallery-item">
    <dt class="gallery-icon">
      <a href="https://www.uelectronics.info/wp-content/uploads/2014/11/asm3.png" title="ZX Spectrum in your browser" rel="gallery1"><img src="https://www.uelectronics.info/wp-content/uploads/2014/11/asm3-150x150.png" width="150" height="150" alt="ZX Spectrum in your browser" /></a>
    </dt>
    
    <dd class="gallery-caption" id="caption451">
      <span class="imagecaption">ZX Spectrum in your browser</span><br />
    </dd>
  </dl>
  
  <dl class="gallery-item">
    <dt class="gallery-icon">
      <a href="https://www.uelectronics.info/wp-content/uploads/2014/11/asm5.png" title="Manic on the PMD-85 emulator" rel="gallery1"><img src="https://www.uelectronics.info/wp-content/uploads/2014/11/asm5-150x150.png" width="150" height="150" alt="Manic on the PMD-85 emulator" /></a>
    </dt>
    
    <dd class="gallery-caption" id="caption449">
      <span class="imagecaption">Manic on the PMD-85 emulator</span><br />
    </dd>
  </dl>
  
  <br style='clear: both' />
</div>