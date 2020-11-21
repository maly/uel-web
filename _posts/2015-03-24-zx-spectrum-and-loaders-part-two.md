---
id: 475
title: ZX Spectrum and loaders – part two
date: 2015-03-24T15:31:02+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=475
permalink: /2015/03/24/zx-spectrum-and-loaders-part-two/
image: /wp-content/uploads/2015/03/zxsloader2.png
categories:
  - My projects
  - Programming
  - Z80
tags:
  - assembler
  - loader
  - Z80
  - ZX Spectrum
---
I unexpectedly discovered one nice piece about loaders which I would like to share. It&#8217;s not very impressive and its magic is hidden inside, but who knows, maybe someone will find it useful.

<!--more-->

While elsewhere we often go from theory to practice, let&#8217;s do it the other way around this time and go straight to practice. Here&#8217;s [a .tzx file](http://retrocip.cz/zxs/games/loader.tzx), try to run it in the emulator&#8230;

(a necessary break to download the file, start the emulator, start loading&#8230; wait a bit&#8230; well&#8230; hmm&#8230; and what is this, is this all?)

Yeah, that&#8217;s all. It is just screens loading from tape. I told you, there&#8217;s not much of an effect. But try to look at the TAP file. _Sure, it&#8217;s BASIC, loader, individual screens, yep, 2682 bytes, 2189, 3340, 4522 bytes, ah&#8230; they are compressed, but they are being loaded directly&#8230; the loader must be decompressing them on the fly!_

Well there you go!

In [the previous article](https://www.uelectronics.info/2015/03/21/zx-spectrum-and-loaders-part-one/ "ZX Spectrum and loaders – part one") I dandily claimed that [DigiSynth](http://www.worldofspectrum.org/infoseekid.cgi?id=0023689) was unpacking data during loading. _Wow, really? Wasn&#8217;t I just dreaming that? I do have a leaky memory after all&#8230;_ So I downloaded [DigiSynth](http://www.worldofspectrum.org/infoseekid.cgi?id=0023689), stared into the code for a while, and then I saw a familiar part: No, I wasn&#8217;t dreaming! Good, I wanted to let it go, but you know how this goes&#8230; In a subway, I started tinkering: After all, it cannot be that difficult to reconstruct the loader and make a packer for it &#8230; and Huffman compression is simple enough&#8230;

## Huffman compression

&#8230;is really simple. At least once you know a bit about compression methods. If not, let me give you a quick 101:

The simplest compression methods are those which eliminate long sequences of bytes of the same value (RLE). They are quick and easy, so they can be used in a loader and deployed into a copying program (so you can fit more stuff into a memory because during loading it compresses data simply like this: &#8220;The following block contains sixty zeros!&#8221;)

Better compression methods exploit the fact that some sequences are often repeated. They therefore create a dictionary of repeating sequences, and replace those sequences with short codes. That is why they are called &#8220;dictionary methods&#8221;. They are based on ancient dictionary algorithms, namely LZ (Lempel-Ziv).

Mr. Huffman have chosen [another approach](http://cs.wikipedia.org/wiki/Huffmanovo_k%C3%B3dov%C3%A1n%C3%AD), he suggested compression based not on sequences, but the frequency of occurrence of certain values. In short, it takes all values in the file (eg bytes, so 0 to 255) and count how many times the value occurs in the input file. Using this information it creates a code for each value (a sequence of bits), which has a property that the more frequent the value, the shorter the sequence. For example, those screens have very often zeroes in them. If there really is a frequent occurence of a zero value, it is encoded into some two bits. Even into one, in an extreme case. Yes, on the other hand, the less frequent values can easily occupy twelve or fifteen bits. But this loss is more than made up for by those frequent values.

If the input file contains different values and all of them have approximately the same count, then Huffman compression becomes ineffective, but it gives back good results for regular files. It is also often used to compress the values of LZ compression dictionary (LZHUF). It is also used in JPEG algorithm&#8230;

I will not go into the implementation details, it&#8217;s enough to know that the algorithm creates a binary tree, which eventually has exactly the same property that I described above.

The disadvantage is that we need to decipher this binary tree first. This disadvantage can be bypassed by adaptive Huffman coding, but that&#8217;s computationally demanding during decompression. And therefore not very suitable for Spectrum loaders&#8230;

Decompression tree as it is implemented, is in principle a very simple structure. Each item has two values, one for a zero bit, the other for a one bit. The value is either a reference to another item (if the code continues), or the resulting number.

### Illustratively

Suppose we have a string ABRAKADABRA. You can see [the tree here](https://atrey.karlin.mff.cuni.cz/~tnovak/compress/huffman/), and the resulting code is:

A: 0  
B: 100  
R: 11  
K: 1011  
D: 1010

The decompression tree will therefore look like this:

<pre class="">0: [*, A | ., 1]
1: [., 2 | *, R]
2: [*, B | ., 3]
3: [*, D | *, K]</pre>

Do not worry, I will explain in a jiffy. Decompression always starts on the first item. If the first bit is zero, the left part is taken (*, A); if the bit is a one, the right part is taken (., 1). The asterisk means &#8220;I already know this character, it&#8217;s this one!&#8221; Dot means &#8220;continue with that entry&#8221;.

Suppose incoming bits will be 0, 1, 0, 0, 1, 1, 0, &#8230; What will happen?

  * Start with record number 0.
  * Bit=0: the left side is telling us that we have found the character A. We have a first byte and we are starting again with record #0
  * Bit=1: the right part says that we should continue with record #1
  * Bit=0: the left side (.,2) says that we should continue with record #2
  * Bit=0: the left part of the record 2 says that we have found letter B. We have a second byte and We are starting again with record #0
  * Bit=1: the right part says that we should continue with record #1
  * Bit=1: the right part says that we found a character R. Again begin from record #0
  * Bit=0: the left part says that we have found a character A.
  * &#8230; And so on.

## Implementation

I wrote the compression algorithm in JavaScript. [You can use it yourself](http://retrocip.cz/zxs/games/huffman.html), it is a standard HTML page, where you use drag and drop to transfer the file you want to compress, and it returns .tap file with the result, suitable for the loader. _Warning: IE3 running on Pentium MMX will probably not work. It doesn&#8217;t even work with new IE. Use Chrome or Firefox, thanks._

The resulting file has the following format:

  1. A flag byte. I am ignoring it
  2. Checksum. XOR of all the values of the input file
  3. Length of the decompression table (number of records)
  4. Data for decompression table. Each entry is stored as a 2×9 bits. The first bit is an attribute, the following eight bits are a value. The attribute determines whether it is a target value (1) or a reference (0). The first 9 bits is for a zero bit, the remaining 9 for a one bit.
  5. The length of the file in bytes. 2 bytes.
  6. Compressed file as bitstream
  7. Bit alignment to eightsome, so there are no problems when copying files

At the beginning the custom loader is a copy of a standard ROM loader, as [described previously](https://www.uelectronics.info/2015/03/21/zx-spectrum-and-loaders-part-one/ "ZX Spectrum and loaders – part one"). To retrieve the entire byte I am using a slightly modified LD\_8\_BITS routine, which does not put the result into the register L, but instead into the registry E. LD_EDGE is not part of the routine, I&#8217;m calling those from the ROM (because I do not create any special effect, additionaly, emulators work better like this and allow for various accelerated loading).

For the decompression table we need to find 1kB of space from the address, which is aligned to the value of $400. I chose $FC00, but you can select a different one. When storing, the attribute of value/reference is stored as a whole byte (00/FF), testing is then simpler (by using simple rotation either zero or one is copied into CY register).

The routine does not check the flag byte nor the data length, all it needs is the address for storing data in the registry IX.

The routine uses no special tricks, everything is as straightforward as I described above.

Oh, and if you want, you can use it for your own creations, it is licensed under CC-0 (Public Domain) licence. Direct your thanks to the author of the original routine from [DigiSynth](http://www.worldofspectrum.org/infoseekid.cgi?id=0023689)&#8230;

Of course, it is possible to improve the compression, remove repetitive sequences, precompress, thereby resulting in improved compression ratio. However, my goal wasn&#8217;t a beefy compressor, but to show you how you can incorporate an interesting functionality into a loader.

PS: [Manic Miner, the Huffman way](http://retrocip.cz/zxs/games/hmanic.tzx)

<pre class="lang:asm decode:true">.ORG    $f000 ;61440
          .ENGINE zxs 
 
                  ;loader test
AGAIN:            
          LD      ix,$4000 
          CALL    LD_BYTES 
          JP      again 
 
                  ;---- more or less copies of RAM routines - ignoring flag byte and attributes
 
LD_BYTES:         
          DI      ;disable interruption
          LD      A,$0F ;white BORDER.
          OUT     ($FE),A 
          LD      HL,$053F ;Adresss SA/LD_RET
          PUSH    HL ;into buffer
          IN      A,($FE) ;Test $FE gate
          RRA     ;Rotation of read byte
          AND     $20 ;bytes, but cosidering just EAR bit
          OR      $02 ;Signal BORDER red is also stored in
          LD      C,A ;register C. ($22 for OFF and $02 for ON state of the EAR input)
          CP      A ;zero flag is set to 1.
 
                  ;First task during loading is to determine
                  ;whether a pulse signal exists (therefore edges
                  ;on-off and off-on).
 
LD_BREAK:         
          RET     NZ ;return from BREAK.
LD_START:         
          CALL    LD_EDGE_1 ;if there is no signal during 1400 T
          JR      NC,LD_BREAK ;return with CY=1.
                  ;otherwise BORDER is set to cyan.
 
                  ;next we wait and check for signal presence
 
          LD      HL,$0415 ;the wait period is nearly a second 
LD_WAIT:          
          DJNZ    LD_WAIT 
          DEC     HL 
          LD      A,H 
          OR      L 
          JR      NZ,LD_WAIT ;waiting loop
          CALL    LD_EDGE_2 ;continue when catching two subseqent
          JR      NC,LD_BREAK ;edges in current period
 
                  ;now just the loading signal will be accepted
 
LD_LEADER:        
          LD      B,$9C ;timing constant
          CALL    LD_EDGE_2 ;continue when catching two subsequent
          JR      NC,LD_BREAK ;edges in current period
          LD      A,$C6 ;these edges must be caught during
          CP      B ;3000 T.
          JR      NC,LD_START 
          INC     H ;number of pairs of edges is stored into H
          JR      NZ,LD_LEADER ;until there is 256 of them
 
                  ;parts off and on of pulse sync come after boot signal 
 
LD_SYNC:          
          LD      B,$C9 ;timing constant.
          CALL    LD_EDGE_1 ;every edge is checked
          JR      NC,LD_BREAK ;until two edges are found close to each other
          LD      A,B ;(starting sync pulse).
          CP      $D4 
          JR      NC,LD_SYNC 
          CALL    LD_EDGE_1 ;at the end there must be a final edge of the on part 
          RET     NC ;of the sync pulse
 
                  ;now header or program bytes can be loaded
                  ;during operations LOAD, VERIFY.
                  ;the first byte defines a type 
 
          LD      A,C ;BORDER to green / magenta
          XOR     $06 
          LD      C,A 
 
                  ;---------
                  ; this is where actual loading begins
                  ; first the decompression tree gets created
                  ; it is stored from address FC00 (must be divisible by $400)
                  ; constant HUF_TABLE is this address / $400
 
HUF_TABLE EQU     $3F ; $3F * $400 =&gt; $FC00
 
          LD      hl,HUF_TABLE * $400 
          CALL    LD_byte ;flag byte -&gt; E
          RET     nc 
                  ;throw away flag
          CALL    LD_byte ;checksum -&gt; E
          RET     nc 
                  ;store checksum into A'
          LD      a,e 
          EX      af,af' 
          CALL    LD_byte ; number of quartlets in compression tree
          RET     nc 
          LD      d,e 
                  ; D values for table
HUF_DECODE:       
          LD      B,$B2 
          CALL    LD_EDGE_2 ;locate length of pulses of each bit
          RET     NC ;return if wrong (longer) pulse length (then CY=0)
          LD      A,$CB ;compare length to about 2400 T,
          CP      B ;when for a zero bit is CY=0 and for a one bit is CY=1.
                  ; first bit of recording is an attribute value / reference
          SBC     a,a ; CY=0 -&gt; 00, CY=1 -&gt; FF
          LD      (hl),a ; store into memory attribute as 00 or FF
          INC     hl ; after it the first value / reference
          CALL    ld_byte ; load a byte
          LD      (hl),e ; and store
          INC     hl ; both will repeat for a one bit
          LD      B,$AF 
          CALL    LD_EDGE_2 ;locate length of pulses of individual bits
          RET     NC ;return if wrong (longer) pulse length (then CY=0)
          LD      A,$CB ;compare length to about 2400 T,
          CP      B ;when for a zero bit is CY=0 and for a one bit is CY=1.
                  ; one bit of attribute
          SBC     a,a 
          LD      (hl),a 
          INC     hl 
          CALL    ld_byte 
          LD      (hl),e 
          INC     hl ; quartlet done
          DEC     d ; already a completed tree?
          JR      nz,huf_decode ; not yet, keep reading
 
                  ; table is ready, we can load data now
                  ; first length into registers DE
          CALL    ld_byte 
          LD      h,e 
          CALL    ld_byte 
          LD      d,e 
          LD      e,h 
 
          LD      A,C ;BORDER to blue and yellow
          XOR     $05 
          LD      C,A 
 
                  ; main loading loop
HUF_LOAD:         
          LD      b,$b2 
          LD      l,0 
HUF_BIT:          
          CALL    LD_EDGE_2 
          RET     nc 
          LD      a,b 
          CP      $cc ; slightly modified constant
          LD      h,HUF_TABLE ; H is an upper byte of address / 4
                  ; L lower (reference to a record)
          CCF     
          ADC     hl,hl 
          ADD     hl,hl 
 
                  ; HL = table address * 4 + 2 * CY
                  ; therefore for CY=0 it is 4*HL, for CY=1 it is 4*HL+2
 
          RRC     (hl) ; attribute value/reference into CY
          INC     hl 
          LD      l,(hl) ; in L is now value or reference
          LD      b,$b1 ; set up a timing constant
          JR      nc,huf_bit ; if it was a reference, continue with next bit
          LD      (ix+0],l ; if not, in L is a value to store
          INC     ix ; address++
          DEC     de ; counter--
          EX      af,af' 
          XOR     l ; checksum in A'
          EX      af,af' 
          LD      a,d ; all bytes read?
          OR      e 
 
          JR      nz,huf_load ; not yet!
 
          EX      af,af' 
          CP      01 ; if A' not zero, then CY=0 - therefore an error
 
          RET     
 
                  ; byte load into E
 
LD_BYTE:          
          LD      B,$B2 ;timing constant.
LD_MARKER:        
          LD      E,$01 ;storing of a marker bit 
 
                  ;this loop combines the loading byte into registry E
 
LD_8_BITS:        
          CALL    LD_EDGE_2 ;locate length of pulses of each bit
          RET     NC ;return if wrong (longer) pulse length (then CY=0)
          LD      A,$CB ;compare length to about 2400 T,
          CP      B ;when for a zero bit is CY=0 and for a one bit is CY=1.
          RL      E ;storing of a new bit into registry E.
          LD      B,$B0 ;timing constant for next bit
          JR      NC,LD_8_BITS ;it was not the last 8th bit
                  ;jump back into the loop.
                  ; 
          RET     ;return with CY=1
 
LD_EDGE_2 EQU     $05e3 
LD_EDGE_1 EQU     $05e7 
</pre>

<small>(Translated by <a href="https://plus.google.com/+Ond%C5%99ejFicek/posts">Ondřej Ficek</a>)</small>