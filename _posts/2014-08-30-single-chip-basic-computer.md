---
id: 417
title: Single chip BASIC computer
date: 2014-08-30T15:12:00+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=417
permalink: /2014/08/30/single-chip-basic-computer/
categories:
  - AVR
---
A [computer running BASIC](http://hackaday.io/project/2428), generating composite video and reading PS/2 keyboard input using a single ATmega microcontroller. A computer running the TinyBASIC programming language on an ATmega 1284P microcontroller as well as generating composite video signals and reading PS/2 keyboard input. The computer is easy to assemble at home as all components are through-hole. [You can buy ready made PCB](http://rover.ebay.com/rover/1/711-53200-19255-0/1?icep_ff3=2&pub=5575085282&toolid=10001&campid=5337554641&customid=&icep_item=121410644845&ipn=psmain&icep_vectorid=229466&kwid=902099&mtid=824&kw=lg). System features include:

  * 8bit ATmega microcontroller running at 16MHz (ATmega 1284P).
  * Over 7KB of memory available for creating BASIC programs.
  * Header for external EEPROM &#8216;cards&#8217; which allow full size programs to be saved (using a 25LC640 EEPROM IC); 4KB internal EEPROM available within the microcontroller.
  * USBasp programming header allowing easy programming of the computer once assembled.
  * Many GPIO pins for connecting to components and other circuits.