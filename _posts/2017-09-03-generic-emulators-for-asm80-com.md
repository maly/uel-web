---
id: 2258
title: Generic Emulators for ASM80.com
date: 2017-09-03T14:19:54+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=2258
permalink: /2017/09/03/generic-emulators-for-asm80-com/
categories:
  - "6502"
  - Microprocessors
  - Programming
  - Z80
tags:
  - ASM80
  - emulator
---
Jeffrey Henning asked me some time ago about &#8220;barebone emulator&#8221; for [ASM80.com](https://www.asm80.com/). Here is my answer&#8230;

<!--more-->

Barebone emulator, or &#8220;generic emulator&#8221;, is a customizable emulator of CPU + memory + serial terminal. You can configure it by simple config file. Let&#8217;s long story short&#8230;

First of all, prepare a config file, e.g. &#8220;mycomputer.emu&#8221;. Extension &#8220;.emu&#8221; is mandatory. File should contain this:

<pre class="lang:default decode:true">cpu Z80

memory.ram.from 0x1000
memory.ram.to 0x13ff
memory.rom.from 0x0000
memory.rom.to 0x0fff

;serial simple
;serial.in 1
;serial.out 1
;serial.status 0
;serial.status.available 0x20
;serial.status.ready 0x02

serial 6850
serial.data 0x81
serial.control 0x80

terminal.caps 1</pre>

Format is really simple: Each line contains one directive. Lines with &#8220;;&#8221; at the same beginning are comments. Here are parameters:

<table>
  <tr>
    <td>
      cpu
    </td>
    
    <td>
      can be &#8220;I8080&#8221;, &#8220;Z80&#8221; or &#8220;C6502&#8221;
    </td>
  </tr>
  
  <tr>
    <td>
      memory.ram.from
    </td>
    
    <td>
      starting address for RAM (please follow the 0x&#8230; convention for hexadecimal numbers)
    </td>
  </tr>
  
  <tr>
    <td>
      memory.ram.to
    </td>
    
    <td>
      last address of RAM
    </td>
  </tr>
  
  <tr>
    <td>
      memory.rom.from,<br /> memory.rom.to
    </td>
    
    <td>
      Same as above, but for ROM (your source code should lays here)
    </td>
  </tr>
  
  <tr>
    <td>
      serial
    </td>
    
    <td>
      Type of serial port. You can select &#8220;6850&#8221; or &#8220;simple&#8221;. See below
    </td>
  </tr>
  
  <tr>
    <td>
      serial.data,<br /> serial.control
    </td>
    
    <td>
      For 6850 serial &#8211; two ports.
    </td>
  </tr>
  
  <tr>
    <td>
      serial.in,<br /> serial.out
    </td>
    
    <td>
      in and out ports for &#8220;simple serial port&#8221;
    </td>
  </tr>
  
  <tr>
    <td>
      serial.status
    </td>
    
    <td>
      &#8220;Simple port&#8221; status. Read only port. It returns &#8220;serial.status.available&#8221; when there is any data for read, and &#8220;serial.status.ready&#8221; if port is ready to send a data.
    </td>
  </tr>
  
  <tr>
    <td>
      terminal.caps
    </td>
    
    <td>
      1 sets terminal to caps lock (default value 0 means &#8220;no caps&#8221;)
    </td>
  </tr>
  
  <tr>
    <td>
      serial.interrupt
    </td>
    
    <td>
      value 1 means that terminal invoke CPU interrupt on keypress (=serial data is ready for read). Default 0 means &#8220;no interrupt&#8221;.
    </td>
  </tr>
</table>

Two serial ports are available &#8211; 6850 is the standard ACIA circuit, &#8220;simple&#8221; is a generic serial port with no complicated functions, just read and send bytes.

How to use it? It&#8217;s simple, just tell which emulator should use by the &#8220;.engine&#8221; directive (without &#8220;.emu&#8221;), like this:

<pre class="lang:asm decode:true ">org 0
.engine mycomputer

    di
    ld sp, 0x1400
    ld a, 15h
    out (80h),a
    ld hl, hello
    call printhl

endless:
    call geta
    jr z,endless
    inc a
    call printa
    
    jr endless
    
printhl: 
    ld a,(hl)
    or a
    ret z
    call printa
    inc hl
    jr printhl
    
printa: 
    push af
serdy:  
    in a,80h
    and 02
    jr z,serdy
    pop af
    out 81h,a
    ret
    
geta:
    in a,80h
    and 01h
    ret z
    in a,81h
    or a
    ret
    
hello:
    db "Hello World!",0x0d,0x0a,0</pre>

It&#8217;s a simple &#8220;hello&#8221; program, using the above configuration. Just save it as &#8220;test.z80&#8221; and click to &#8220;Emulate (F10)&#8221;. It should compile and run emulator with given configuration.

For 6502-based computer you can use this config file (e.g.&#8221;my6502.emu&#8221;):

<pre class="lang:default decode:true">cpu C6502

memory.ram.from 0x0000
memory.ram.to 0x0fff
memory.rom.from 0xf000
memory.rom.to 0xffff

serial 6850
serial.data 0xa001
serial.control 0xa000
serial.map 1
</pre>

Please notice the &#8220;serial.map&#8221; directive. It means that serial port is not mapped into &#8220;I/O&#8221; space (like Z80/8080 does), but into memory space. Serial.data and serial.control are addresses now.

6502 requires RAM in a bottom part of address space, ROM at the top.

Try this code:

<pre class="lang:asm decode:true ">.engine my6502

ACIA         =  $A000 
ACIACONTROL  =  ACIA+0 
ACIASTATUS   =  ACIA+0 
ACIADATA     =  ACIA+1 

          .ORG    $FFFC 
          DW      reset 
          DW      reset
          
.org $f000

RESET:          
          LDX     #$FF 
          TXS     
          LDA     #$15
          STA     ACIAControl 
 
          LDY     #0 
LOOP:     
          LDA     Message,Y
          BEQ     KEY
          JSR     SEROUT
          INY
          BNE     LOOP
KEY:      LDA     ACIAStatus
          AND     #1
          BEQ     KEY
          LDA     ACIAData 
; simple data mangling
          SEC
          ADC #0
          JSR     serout 
 
          JMP     KEY
 
Message:   
          DB      $0C,"My hovercraft is full of eels!",$0D,$0A,$00 
 
SEROUT:   PHA
SO_WAIT:  LDA     ACIAStatus
          AND     #2
          BEQ     SO_WAIT
          PLA
          STA     ACIAData
          RTS</pre>

Other processors and peripherals are on its way&#8230;