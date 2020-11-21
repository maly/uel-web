---
id: 480
title: 'ZX Spectrum and loaders &#8211; part three'
date: 2015-03-27T09:21:00+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=480
permalink: /2015/03/27/zx-spectrum-and-loaders-part-three/
image: /wp-content/uploads/2015/03/screenshot-www.youtube.com-2015-03-25-20-02-19.png
categories:
  - My projects
  - Programming
  - Z80
tags:
  - assembler
  - loader
  - Sinclair
  - Z80
  - ZX Spectrum
---
This is a continuation of an earlier [article about ZX Spectrum loaders](https://www.uelectronics.info/2015/03/21/zx-spectrum-and-loaders-part-one/). Hold on to your hats and get your calculators ready, we are going downhill! ([Part 1](https://www.uelectronics.info/2015/03/21/zx-spectrum-and-loaders-part-one/), [Part 2](https://www.uelectronics.info/2015/03/24/zx-spectrum-and-loaders-part-two/ "ZX Spectrum and loaders – part two"))

<!--more-->

[Last time](https://www.uelectronics.info/2015/03/21/zx-spectrum-and-loaders-part-one/ "ZX Spectrum and loaders – part one") I finished up stating that next time we&#8217;ll need to count and weigh each and every processor tick. This time, our closest ally will be a waiting loop in a routine LD\_EDGE\_1. Let&#8217;s recall that routine:

<pre class="lang:default decode:true ">LD_EDGE_2:
          CALL    LD_EDGE_1 ;calling, in fact, LD_EDGE_1 once more
          RET     NC ;return if error
LD_EDGE_1:
          LD      A,$16 ;7T
LD_DELAY:
          DEC     A ;4T
          JR      NZ,LD_DELAY ;12T / 7T
          AND     A ;4T

LD_SAMPLE:</pre>

Do you see it there in LD_DELAY? Yes? So that&#8217;s it right there! It takes 7T + 21*(4T+12T) + (4T+7T) = 354T, which is quite a lot of ticks for our purposes. Which purposes, you ask? Well, mostly likely a counter for how much data is remaining to be loaded.

## Graphical indicator

In my text game Poradce (The Consultant) I used, besides for some scary flashing, some kind of a &#8220;thermometer&#8221; that showed how much remains to be loaded. It was that little something on the left side of the screen&#8230;



This effect is simpler than you might have expected. The length of the column is almost exactly &#8220;file length / 512&#8221; (I say &#8220;almost exactly&#8221; because I had painted it a little shorter&#8230;) So I&#8217;ll take the upper byte of remaining number of bytes (which is in register pair DE), divide it by two, add offset from the bottom of the screen, convert the coordinate to an address on the screen and mask the byte with value %11100111 &#8211; therefore everything else remains except for just two points in the middle, which, coincidentaly, exactly represent our &#8220;thermometer&#8221; and they get overwritten. To recalculate, I even use a routine PIXEL-ADD ($22AA) from Spectrum&#8217;s ROM, which takes the coordinates in B and C (the same coordinate system the PLOT has) and calculates an address on the screen (HL) and a mask of the point (A).

<pre class="lang:default decode:true">LD_EDGE_2:        
          CALL    LD_EDGE_1 
          RET     NC 
LD_EDGE_1:        
          JP      LD_OWN 
LD_BACK:          
          LD      A,03 ;7T
LD_DELAY:         
          DEC     A ;4T
          JR      NZ,LD_DELAY ;12T/7T
                  ;7T + 2*(4+12) + (4+7) = 50T
          AND     A 
LD_SAMPLE:        
          INC     B 
          RET     Z 
          LD      A,7f 
          IN      A,(fe) 
          RRA     
          RET     NC 
          XOR     C 
          AND     20 
          JR      Z,LD_SAMPLE 
          LD      A,C 
          CPL     
          LD      C,A 
          LD      A,06 
          RRCA    
          AND     07 
          OR      08 
          OUT     (fe),A 
          SCF     
          RET     

LD_OWN:           
          PUSH    DE ;11T
          PUSH    HL ;11T
          PUSH    BC ;11T
          PUSH    AF ;11T
          LD      B,D ;4T
          SRL     B ;8T
          NOP     ;4T
          INC     B ;4T
          INC     B ;4T
          LD      C,00 ;7T
          CALL    22aa ;17T + 132T routine
          LD      A,(HL) ;7T
          AND     e7 ;7T
          LD      (HL),A ;7T
          POP     AF ;10T
          POP     BC ;10T
          POP     HL ;10T
          POP     DE ;10T
          JP      LD_BACK ;10T
</pre>

As you can see, I interfered with the timing loop. At the beginning I popped out into my own routine which operates that effect (it is 305T with that diversion) and after returning back I still have enough time, so I&#8217;ll wait another 50T, which translates to 355T, which it is pretty much exactly our magic number!

## Numeric indicator

If you rather want to show a number instead of some dwindling column, things get more difficult. Each digit must in fact be drawn on the screen, which means eight writes for each digit. And for a three-digit counter this is quite a lot of time, it won&#8217;t fit into 354T. So you need to chop the algorithm into parts that fit within this time limit and call them in sequence. The second set of registers and index register IY will be your invaluable tool for this.

Games from Hewson and Czech programs from Universum (I think) had some very nice counters, they used their own font and they also shown fliping digits effect. In another text game I used just a simple counter of &#8220;the number of bytes / 64&#8221;, which for 48kB block still fits into three digits. At the beginning I prepared the desired digits (in decimal!) into registers D and E (resp. into the mirror ones):

<pre class="lang:asm decode:true">PUSH    DE
          EXX
          POP     HL
          LD      DE,00
          LD      BC,$40
LD_DIV:
          AND     A
          SBC     HL,BC
          JR      C,b0be
          LD      A,E
          ADD     A,01
          DAA
          LD      E,A
          LD      A,D
          ADC     A,00
          DAA
          LD      D,A
          JR      LD_DIV
LD_DIV2:
          LD      H,3d
          LD      BC,4001
          EXX
          LD      IY, LD_CHARS ; coming up next...</pre>

This effect did not take place directly in LD\_EDGE but in LD\_8\_BITS instead. However, LD\_EDGE has been modified so as to use significantly different timing values during loading:

<pre class="lang:asm decode:true">LD      B,b0
          LD      L,01
LD_8_BITS:
          CALL    LD_EDGE_2M
          RET     NC
          LD      A,d4
          CP      B
          RL      L
          LD      B,b0
          JP      NC,LD_LONGWAY
          LD      A,H
          XOR     L
          LD      H,A
;... the rest of it is normal...

;LD_EDGE_2M : 468 + 58 * B
;shortened by 462T
LD_EDGE_2M:
          CALL    LD_EDGE_1M
          RET     NC
          LD      A,10
          DEC     A
          JR      NZ,$-1 ;  254T
LD_EDGE_1M:
          AND     A
          INC     B
          RET     Z
          LD      A,7f
          IN      A,(fe)
          RRA
          XOR     C
          AND     20
          JR      Z,b15b
          LD      A,C
          CPL
          LD      C,A
          AND     04
          OR      08
          OUT     (fe),A
          SCF
          RET
</pre>

The effect ifself then played out as follows:

<pre class="lang:asm decode:true ">LD_LONGWAY:
          EXX
          DEC     C
          JP      Z,LD_SUB1
          JP      (IY)

LD_RETHERE:
          POP     IY
          EXX
          JP      LD_8_BITS

LD_SUB1:
          LD      C,07
          DEC     B
          JP      Z,LD_SUB2
          LD      A,04
          DEC     A
          JR      NZ,b1a9 ;wait 59T
          EXX
          JP      LD_8_BITS

LD_SUB2:
          LD      A,E
          SUB     01
          DAA
          LD      E,A
          LD      A,D
          SBC     A,B ;in B there is 0
          DAA
          LD      D,A
          LD      B,$40
          LD      IY,LD_CHARS
          LD      HL,$3d00
          EXX
          JP      LD_8_BITS

                  ;1st digit
LD_CHARS:
          LD      A,E
          ADD     A,A
          ADD     A,A
          ADD     A,A
          OR      $81
          LD      L,A
          LD      A,(HL)
          LD      (50fd),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (51fd),A
          INC     L
          LD      A,(HL)
          LD      (52fd),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (53fd),A
          INC     L
          LD      A,(HL)
          LD      (54fd),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (55fd),A
          INC     L
          LD      A,(HL)
          LD      (56fd),A
          CALL    LD_RETHERE

                  ;2nd digit
          LD      A,E
          AND     $f0
          RRA
          OR      $81
          LD      L,A
          LD      A,(HL)
          LD      (50fc),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (51fc),A
          INC     L
          LD      A,(HL)
          LD      (52fc),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (53fc),A
          INC     L
          LD      A,(HL)
          LD      (54fc),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (55fc),A
          INC     L
          LD      A,(HL)
          LD      (56fc),A
          CALL    LD_RETHERE

                  ;3rd digit
          LD      A,D
          ADD     A,A
          ADD     A,A
          ADD     A,A
          OR      $81
          LD      L,A
          LD      A,(HL)
          LD      (50fb),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (51fb),A
          INC     L
          LD      A,(HL)
          LD      (52fb),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (53fb),A
          INC     L
          LD      A,(HL)
          LD      (54fb),A
          CALL    LD_RETHERE
          INC     L
          LD      A,(HL)
          LD      (55fb),A
          INC     L
          LD      A,(HL)
          LD      (56fb),A
          LD      (56fb),A
          NOP
          LD      IY,LD_CHARS
          EXX
          JP      LD_8_BITS
</pre>

This, of course, deserves a few explanatory notes:

Registers DE contain the counter itself, stored in BCD coding. The register C contains a bits counter, register B contains a bytes counter. Once they calculate down to zero, the counter gets decreased by 1, the result gets adjusted by DAA instruction and prepared for the actual printing of characters. HL holds the value of $3D00, which coincidentally is an address in the ROM where numbers are stored, and IY holds an address of the routine LD_CHARS, which displays the digits.

When you chop the routine into pieces, you are left with two options. You either edit the code and rewrite the address where you need to jump next, or you store the address somehow. Here it is in the IY register. By using a simple trick CALL LD\_RETHERE (as in RETurn HERE) you go back to LD\_8\_BITS, while the return address gets into IY, so that during the next call the JP (IY) will be directed to the next piece. Note that printing of characters is chopped into small pieces by calling LD\_RETHERE.

## Time indicator and more

Calculating remaining time is more complicated than counting bytes. You either have to count every bit differently (a one bit is twice as long as a zero bit), or use an interrupt (yes, interrupt in loader!)

I was looking for an example of the first approach, but before I could comment on it, [Busy](http://www.worldofspectrum.org/infoseekpub.cgi?regexp=^Busy+Software$) called me and said that he had found his [Overscan loader source code](http://busy.speccy.cz/tvorba/overscan.htm), asking me if I want to publish it. So here we go, with Busy&#8217;s courtesy, Overscan loader!

(Time calculation and graphical effects are included!)

  
And here is the source code with Busy&#8217;s comments (Thanks!) Because it is a bit long, I shall say my goodbyes right now and wish you good luck in your own experiments.

<pre class="lang:default decode:true">5b00            *a
5b00            *s
5b00            ;===============================================================;
5b00            ;== Version 16 == Loader for Overscan == 12.08.1991 Busy soft ==;
5b00            ;===============================================================;
5b00            znaky  =    #8800	modifiet charset
5b00                   org  #8200,0
8200 f3         p      di
8201 310082            ld   sp,p
8204 fd217f00          ld   iy,#7f	timers init for time and auto kitt
8208 210080            ld   hl,#8000	IM2 vector init
820b 110180            ld   de,#8001
820e 011001            ld   bc,#0110
8211 3681              ld   (hl),#81
8213 edb0              ldir
8215 212183            ld   hl,rut	relocation of operation routine IM2
8218 118181            ld   de,#8181
821b 010800            ld   bc,load-rut
821e edb0              ldir
8220 067f       i      ld   b,#7f
8222 216711            ld   hl,#1167
8225 11d685            ld   de,k
8228 d5                push de
8229 7e         ll14   ld   a,(hl)	random memory inserts
822a ad                xor  l		(to fool an enemy)
822b 12                ld   (de),a
822c 02                ld   (bc),a
822d a9                xor  c
822e 0b                dec  bc
822f 02                ld   (bc),a
8230 0b                dec  bc
8231 13                inc  de
8232 2b                dec  hl
8233 7c                ld   a,h
8234 b5                or   l
8235 20f2              jr   nz,ll14
8237 e1                pop  hl
8238 012a7a            ld   bc,-k
823b edb0              ldir
823d 3e80       rst    ld   a,#80	IM2 launch
823f ed47              ld   i,a
8241 ed5e              im2
8243 fb                ei
8244 cd0c85            call zn		charset conversion
8247 3e48              ld   a,#48
8249 32b784            ld   (n20+2),a
824c 210040            ld   hl,#4000	screen init
824f 110140            ld   de,#4001
8252 010018            ld   bc,#1800
8255 71                ld   (hl),c
8256 edb0              ldir
8258 010603            ld   bc,#0306
825b 71                ld   (hl),c
825c edb0              ldir
825e 21a341     o      ld   hl,#41a3	squares under kitt effect
8261 0e06              ld   c,#06
8263 061a       ll11   ld   b,#1a
8265 e5                push hl
8266 367e       ll10   ld   (hl),#7e
8268 2c                inc  l
8269 10fb              djnz ll10
826b e1                pop  hl
826c 24                inc  h
826d 0d                dec  c
826e 20f3              jr   nz,ll11
8270 210059            ld   hl,#5900	red frame in the middle
8273 011208            ld   bc,#0812
8276 5d                ld   e,l
8277 73         ll12   ld   (hl),e
8278 2c                inc  l
8279 71                ld   (hl),c
827a 7d                ld   a,l
827b c61d              add  a,#1d
827d 6f                ld   l,a
827e 71                ld   (hl),c
827f 2c                inc  l
8280 73                ld   (hl),e
8281 2c                inc  l
8282 10f3              djnz ll12
8284 21e15a            ld   hl,#5ae1
8287 1e44              ld   e,#44
8289 cd0083            call sap
828c 21e158            ld   hl,#58e1
828f cdfe82            call pas
8292 21015a            ld   hl,#5a01
8295 cdfe82            call pas
8298 215f85            ld   hl,firma	texts printout
829b cd0783            call text
829e 217685            ld   hl,demo
82a1 cd0783            call text
82a4 219185            ld   hl,loatim
82a7 cd0783            call text
82aa cd2983            call load		calling the loader
82ad 08                ex   af,af
82ae af                xor  a
82af cdf884            call out		Border 0
82b2 08                ex   af,af
82b3 3839              jr   c,ok		if we loaded a block correctly, then jump
82b5 af         error  xor  a		if error during loading
82b6 210048            ld   hl,#4800	then show message
82b9 77         ll13   ld   (hl),a
82ba 23                inc  hl
82bb cb64              bit  4,h
82bd 28fa              jr   z,ll13
82bf 112548            ld   de,#4825
82c2 212515            ld   hl,#1525
82c5 cd0b83            call txt
82c8 21d285            ld   hl,krik
82cb cd0b83            call txt
82ce 21a585            ld   hl,rew
82d1 cd0783            call text
82d4 21b785            ld   hl,reload
82d7 cd0783            call text
82da af                xor  a
82db 32b784            ld   (n20+2),a
82de 06ff       press  ld   b,#ff	waiting for keypress
82e0 cdcf83            call mmm		we call loader during waiting
82e3 af                xor  a		so our effects still run
82e4 dbfe              in   a,(#fe)
82e6 f6e0              or   #e0
82e8 3c                inc  a
82e9 28f3              jr   z,press
82eb c33d82            jp   rst
82ee
82ee 210058     ok     ld   hl,#5800	if we loaded a block correctly
82f1 110158            ld   de,#5801	erase all attributes
82f4 0603              ld   b,#03
82f6 edb0              ldir
82f8 cde6c3            call 50150       demo decompression
82fb c376e9            jp   59766	demo launch
82fe
82fe 1e12       pas    ld   e,#12	some help subroutines
8300 061e       sap    ld   b,#1e	for frame drawing
8302 73         pp1    ld   (hl),e
8303 2c                inc  l
8304 10fc              djnz pp1
8306 c9                ret
8307
8307 5e         text   ld   e,(hl)	text printout
8308 23                inc  hl		HL = text address
8309 56                ld   d,(hl)	DE = text position on screen
830a 23                inc  hl
830b e5         txt    push hl
830c d5                push de
830d 6e                ld   l,(hl)
830e 2688              ld   h,&gt;znaky
8310 0608              ld   b,#08
8312 7e         xtx    ld   a,(hl)	print of one character
8313 12                ld   (de),a
8314 14                inc  d
8315 24                inc  h
8316 10fa              djnz xtx
8318 d1                pop  de
8319 e1                pop  hl
831a 1c                inc  e
831b cb7e              bit  7,(hl)	text ends with char with set bit7=1
831d 23                inc  hl
831e 28eb              jr   z,txt
8320 c9                ret
8321
8321 08         rut    ex   af,af	routine from interrupt
8322 fd24              inc  yh		YH = timer for kitt effect
8324 fd2c              inc  yl		YL = timer for counting and time printing
8326 08                ex   af,af
8327 fb                ei
8328 c9                ret
8329
8329 af         load   xor  a		THIS IS WHERE LOADER BEGINS!!
832a cdf884            call out
832d 211f40     rohy   ld   hl,#401f	drawing of cut off corners
8330 11ff57            ld   de,#57ff
8333 7b                ld   a,e
8334 77                ld   (hl),a
8335 12                ld   (de),a
8336 d9                exx
8337 010207            ld   bc,#0702
833a 210040            ld   hl,#4000
833d 11e057            ld   de,#57e0
8340 77                ld   (hl),a
8341 12                ld   (de),a
8342 7e         rr1    ld   a,(hl)
8343 24                inc  h
8344 15                dec  d
8345 cb27              sla  a
8347 77                ld   (hl),a
8348 12                ld   (de),a
8349 d9                exx
834a 7e                ld   a,(hl)
834b 24                inc  h
834c 15                dec  d
834d cb3f              srl  a
834f 77                ld   (hl),a
8350 12                ld   (de),a
8351 d9                exx
8352 10ee              djnz rr1
8354            nnn
8354 3ef8       rrr    ld   a,#f8	catching the leader tone
8356 32f484            ld   (xor+1),a
8359 2600              ld   h,#00
835b 06ff       djnz   ld   b,#ff
835d cdcf83            call mmm
8360 25                dec  h
8361 20f8              jr   nz,djnz
8363 cdcf83            call mmm
8366 30ec              jr   nc,nnn
8368 cdcb83            call ppp
836b 30e7              jr   nc,nnn
836d 069c       sss    ld   b,#9c
836f cdcb83            call ppp
8372 30e0              jr   nc,nnn
8374 3ec6              ld   a,#c6
8376 b8                cp   b
8377 30db              jr   nc,rrr
8379 24                inc  h
837a 20f1              jr   nz,sss
837c 3efc              ld   a,#fc
837e 32f484            ld   (xor+1),a
8381 06c9       ttt    ld   b,#c9
8383 cdcf83            call mmm
8386 30cc              jr   nc,nnn
8388 78                ld   a,b
8389 fed4              cp   #d4
838b 30f4              jr   nc,ttt
838d cdcf83            call mmm
8390 d0                ret  nc
8391 79                ld   a,c		leader and sync-pulse OK, we can start loading
8392 ee03              xor  #03
8394 4f                ld   c,a
8395 cdb583            call byte         flagbyte Load
8398 d0                ret  nc
8399
8399 dd21e6c3   loa    ld   ix,50150	demo bytes loading loop
839d 116a24            ld   de,9322
83a0            ;      ld   ix,#4000	during testing of effects in loader
83a0            ;      ld   de,#1b00	the block was loaded onto the screen
83a0 cdb583     loa1   call byte
83a3 d0                ret  nc
83a4 dd7500            ld   (ix+#00),l
83a7 dd23              inc  ix
83a9 1b                dec  de
83aa 7a                ld   a,d
83ab b3                or   e
83ac 20f2              jr   nz,loa1
83ae cdb583            call byte          load parity
83b1 d0                ret  nc
83b2 fe01              cp   #01
83b4 c9                ret
83b5
83b5 06b2       byte   ld   b,#b2	Load of one byte
83b7 2e01              ld   l,#01
83b9 cdcb83     loa9   call ppp
83bc d0                ret  nc
83bd 3ecb       kk     ld   a,#cb
83bf b8                cp   b
83c0 cb15              rl   l
83c2 06b0              ld   b,#b0
83c4 30f3              jr   nc,loa9
83c6 7c                ld   a,h
83c7 ad                xor  l
83c8 67                ld   h,a
83c9 37                scf
83ca c9                ret
83cb
83cb cdcf83     ppp    call mmm		Load of one byte
83ce d0                ret  nc
83cf d9         mmm    exx		    load of one halfperiod
83d0 c3                db   #c3		there are my special subroutines instead of wating loop
83d1 ec83       skok   dw   n1		jump to part of code to execute next
83d3
83d3 7e         n1sub  ld   a,(hl)              /0
83d4 81                add  a,c		time value recalcullation
83d5 fe0a              cp   #0a
83d7 0e00              ld   c,#00
83d9 3802              jr   c,n1aa
83db 0c                inc  c
83dc af                xor  a
83dd 77         n1aa   ld   (hl),a
83de 23                inc  hl                  \50
83df 7e                ld   a,(hl)
83e0 81                add  a,c
83e1 fe06              cp   #06
83e3 0e00              ld   c,#00
83e5 3802              jr   c,n1bb
83e7 0c                inc  c
83e8 af                xor  a
83e9 77         n1bb   ld   (hl),a
83ea 23                inc  hl                  \100
83eb c9                ret                      \110+call=117
83ec
83ec fd7d       n1     ld   a,yl
83ee fe33              cp   51                  \29
83f0 3866              jr   c,n10	Test if a second has passed
83f2 0e01              ld   c,#01	so that we could draw another time value
83f4 fd7d              ld   a,yl
83f6 d632              sub  50
83f8 fd6f              ld   yl,a
83fa 215a85            ld   hl,cas              \76
83fd cdd383            call n1sub
8400 3e04              ld   a,#04
8402 110784            ld   de,n3
8405 181a              jr   jpde                \222 +/-
8407
8407 23         n3     inc  hl		every second we also change
8408 cdd383            call n1sub	an efect in middle third
840b ed5f              ld   a,r
840d 32b984            ld   (udaj+1),a
8410 3a5b85            ld   a,(cas+1)
8413 0f                rrca
8414 9f                sbc  a,a
8415 e60f              and  #0f
8417 f607              or   #07
8419 32cc84            ld   (rot),a
841c 3e05              ld   a,#05
841e 112a84            ld   de,n2
8421 ed53d183   jpde   ld   (skok),de
8425 1ef9              ld   e,#f9
8427 c3e084            jp   wait                \222 +/-
842a
842a 2b         n2     dec  hl
842b cb7e              bit  7,(hl)              \32
842d 2808              jr   z,n2aa	prepare to print one character
842f 3e0e              ld   a,14
8431 01ec83            ld   bc,n1
8434 c3dc84            jp   buduci
8437 e5         n2aa   push hl
8438 6e                ld   l,(hl)
8439 2688              ld   h,&gt;znaky
843b 1650              ld   d,#50
843d 0602              ld   b,#02               /2*88=202
843f 7e         dd1    ld   a,(hl)	print of one character
8440 12                ld   (de),a
8441 24                inc  h
8442 14                inc  d
8443 7e                ld   a,(hl)
8444 12                ld   (de),a
8445 24                inc  h
8446 14                inc  d
8447 7e                ld   a,(hl)
8448 12                ld   (de),a
8449 24                inc  h
844a 14                inc  d
844b 7e                ld   a,(hl)
844c 12                ld   (de),a
844d 24                inc  h
844e 14                inc  d
844f 10ee              djnz dd1
8451 1c                inc  e
8452 e1                pop  hl
8453 3e01              ld   a,#01               \299
8455 c3e084            jp   wait
8458
8458 fd7c       n10    ld   a,yh		Auto-Kitt effect
845a fe02              cp   #02                 \56
845c 3857              jr   c,n20	test if we can go into another phase
845e c3                db   #c3
845f 9584       jump   dw   n12                 \73
8461
8461 21a358     n11    ld   hl,#58a3
8464 3647              ld   (hl),#47
8466 2c         incdec inc  l
8467 7d                ld   a,l
8468 326284            ld   (n11+1),a
846b 216684            ld   hl,incdec           \124
846e febc              cp   #bc
8470 3f                ccf
8471 3802              jr   c,iidd
8473 fea4              cp   #a4
8475 9f         iidd   sbc  a,a                 \151
8476 e601              and  #01
8478 ae                xor  (hl)
8479 77                ld   (hl),a
847a 218484            ld   hl,jr+1
847d 3e03              ld   a,#03
847f ae                xor  (hl)
8480 77                ld   (hl),a
8481 3ea2              ld   a,#a2               \210
8483 1803       jr     jr   #03
8485 329684            ld   (n12+1),a
8488 fd2600            ld   yh,#00
848b 3e02              ld   a,#02
848d 219584            ld   hl,n12
8490 225f84     buduce ld   (jump),hl
8493 184b              jr   wait                \267
8495
8495 216458     n12    ld   hl,#5864            \73
8498 7d                ld   a,l
8499 febc              cp   #bc                 \94
849b 3011              jr   nc,n12bb
849d 014104            ld   bc,#0441            \111
84a0 7e         n12aa  ld   a,(hl)              /47
84a1 b9                cp   c
84a2 3801              jr   c,#01
84a4 3d                dec  a
84a5 77                ld   (hl),a
84a6 2c                inc  l
84a7 10f7              djnz n12aa               \4*47=188
84a9 229684            ld   (n12+1),hl
84ac 1835              jr   end
84ae
84ae 3e08       n12bb  ld   a,#08
84b0 216184            ld   hl,n11
84b3 18db              jr   buduce              /\174
84b5
84b5 210048     n20    ld   hl,#4800	filling the middle third of the screen
84b8 3e33       udaj   ld   a,#33               \82
84ba 772c772c          dw   #2c77,#2c77	8*[INC C : LD (HL),A]
84be 772c772c          dw   #2c77,#2c77
84c2 772c772c          dw   #2c77,#2c77         8*11=88
84c6 772c772c          dw   #2c77,#2c77         \
84ca 2002              jr   nz,n21              /
84cc 0f         rot    rrca
84cd 24                inc  h
84ce cba4       n21    res  4,h
84d0 cbdc              set  3,h
84d2 22b684            ld   (n20+1),hl
84d5 32b984            ld   (udaj+1),a
84d8 3e03              ld   a,3
84da 1804              jr   wait                \68
84dc
84dc ed43d183   buduci ld   (skok),bc	setting of what part of code to call next
84e0 3d         wait   dec  a
84e1 20fd              jr   nz,wait
84e3 d9         end    exx		all code parts last 200 cycles (+/- give or take)
84e4 a7         zzz    and  a		end of my special subroutines
84e5 04         l222   inc  b
84e6 c8                ret  z
84e7 3eff              ld   a,#ff	#FF instead of #7F does not respond to SPACEBAR
84e9 dbfe              in   a,(#fe)
84eb 1f                rra
84ec d0                ret  nc
84ed a9                xor  c
84ee e620              and  #20
84f0 28f3              jr   z,l222
84f2 79                ld   a,c
84f3 eefc       xor    xor  #fc		#FC instead of CPL makes prettier colours
84f5 4f                ld   c,a		red-yellow boot tone and blue-blue bytes
84f6 e607              and  #07
84f8 320058     out    ld   (#5800),a           +52
84fb 321f58            ld   (#581f),a	setting of corner attributes
84fe f608              or   #08
8500 d3fe              out  (#fe),a
8502 e607              and  #07
8504 32e05a            ld   (#5ae0),a
8507 32ff5a            ld   (#5aff),a
850a 37                scf
850b c9                ret
850c
850c 110088     zn     ld   de,znaky	charset conversion
850f d5         ll1    push de		from ROM font makes contures
8510 7b                ld   a,e		also font gets reorganized so that
8511 fe20              cp   ' '		print routines can run faster
8513 3002              jr   nc,#02
8515 c630              add  a,'0'
8517 87                add  a,a
8518 6f                ld   l,a
8519 260f              ld   h,#0f
851b 29                add  hl,hl
851c 29                add  hl,hl
851d cd4985            call read
8520 4f                ld   c,a
8521 23                inc  hl
8522 cd5085            call write
8525 0606              ld   b,#06
8527 cd4985     ll2    call read
852a 4f                ld   c,a
852b 2b                dec  hl
852c cd4985            call read
852f b1                or   c
8530 4f                ld   c,a
8531 23                inc  hl
8532 23                inc  hl
8533 cd5085            call write
8536 10ef              djnz ll2
8538 cd4985            call read
853b 4f                ld   c,a
853c 2b                dec  hl
853d cd4985            call read
8540 23                inc  hl
8541 cd5485            call rit
8544 d1                pop  de
8545 1c                inc  e
8546 20c7              jr   nz,ll1
8548 c9                ret
8549
8549 7e         read   ld   a,(hl)
854a 0f                rrca
854b 0f                rrca
854c b6                or   (hl)
854d 07                rlca
854e b6                or   (hl)
854f c9                ret
8550
8550 cd4985     write  call read
8553 2b                dec  hl
8554 b1         rit    or   c
8555 ae                xor  (hl)
8556 12                ld   (de),a
8557 23                inc  hl
8558 14                inc  d
8559 c9                ret
855a 00000a00   cas    db   0,0,10,0,0	Buffer for time value
            855f 4550       firma  dw   #5045
8561 42757379          db   'Busy software '
            6
            2
            856f 70726573          db   'presen','t'+#80
            8576 8350       demo   dw   #5083
8578 3e3e3e20          db   '&gt;&gt;&gt; THE OVER'
            0
            8584 5343414e          db   'SCAN DEMO &lt;&lt;'
            d
            8590 bc                db   '&lt;'+#80
8591 e250       loatim dw   #50e2
8593 4c6f6164          db   'Loading'
            859a 20636361          db   ' cca 1 min'
            d
            85a4 ae                db   '.'+#80
85a5 8848       rew    dw   #4888
85a7 52657769          db   'Rewing the tape'
            4
            4
            85b6 ac                db   ','+#80
85b7 c448       reload dw   #48c4
85b9 70726573          db   'press any key '
            e
            5
            85c7 616e6420          db   'and reload'
            f
            85d1 ae                db   '.'+#80
85d2 202121a1   krik   db   ' !!','!'+#80
85d6            k
85d6            l      =    k-p		label "l" will be total code length
85d6
85d6                   org  #6000,0	next routines have no use
6000 f3         mrs    di		they were debug routines
6001 ed56              im1
6003 af                xor  a
6004 ed47              ld   i,a
6006 c3b3f4            jp   #f4b3	jump into MRS
6009
6009 cd0c85     zs     call zn		Test of conversion of charset
600c 210088            ld   hl,znaky	printing of all characters
600f 110040            ld   de,#4000	on the screen
6012 010008            ld   bc,#0800
6015 edb0              ldir
6017 c9                ret
6018
6018 3e0c       poke   ld   a,12		sets testing im2 vector into ROM
601a d317              out  (23),a	(Outs for writing into ROM on MB01)
601c 218181            ld   hl,#8181
601f 22ff3a            ld   (#3aff),hl
6022 3e04              ld   a,4
6024 d317              out  (23),a
6026 c9                ret
6027                   end
</pre>

<small>(Translated by <a href="https://plus.google.com/+Ond%C5%99ejFicek/posts">Ondřej Ficek</a>)</small>