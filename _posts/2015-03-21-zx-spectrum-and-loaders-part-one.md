---
id: 468
title: 'ZX Spectrum and loaders &#8211; part one'
date: 2015-03-21T12:33:21+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=468
permalink: /2015/03/21/zx-spectrum-and-loaders-part-one/
image: /wp-content/uploads/2015/03/jetpac_loadingscreen_3.jpg
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
Everyone who had once owned a ZX Spectrum, surely had at least once seen an amazing loading effect and wondered: How are they doing this? So, come and have a read&#8230; It will be a long story and perhaps you might even learn something.

<!--more-->

Because ZX Spectrum didn&#8217;t have any dedicated circuit for storing and retrieving programs to and from a cassette tape (such as PMD), it had to do it purely by software. At the highest level, programs were loaded as a pair of &#8220;header + data&#8221;. In the header file there was information about the file (name, type, length, etc.) and the data file contained actual custom data.

The header file has a length of 17 bytes and a following structure:

<table>
  <tr>
    <th>
      Position
    </th>
    
    <th>
      Length
    </th>
    
    <th>
      Description
    </th>
  </tr>
  
  <tr>
    <td>
    </td>
    
    <td>
      1
    </td>
    
    <td>
      Type (0=program, 1=number array, 2=character array, 3=code)
    </td>
  </tr>
  
  <tr>
    <td>
      1
    </td>
    
    <td>
      10
    </td>
    
    <td>
      Name (right padded with spaces to 10 characters)
    </td>
  </tr>
  
  <tr>
    <td>
      11
    </td>
    
    <td>
      2
    </td>
    
    <td>
      Length of data block
    </td>
  </tr>
  
  <tr>
    <td>
      13
    </td>
    
    <td>
      2
    </td>
    
    <td>
      Parameter 1. Eg. for programs it is a parameter LINE, for &#8216;code&#8217; it is the address of beginning of the code block
    </td>
  </tr>
  
  <tr>
    <td>
      15
    </td>
    
    <td>
      2
    </td>
    
    <td>
      Parameter 2. Eg. for programs it is the beginning of variables
    </td>
  </tr>
</table>

Data block doesn&#8217;t have any such structure, it just contains data.

Let us descend one level: Each block was preceded by one flag byte ($00 for a header, $ff for data) and terminated with a checksum (all bytes, including the flag byte, XORed together). Header was therefore actually 19 bytes long &#8211; it began with 0 and ended with the checksum, but these bytes were &#8220;consumed&#8221; by the system and you couldn&#8217;t get your hands on them.

This is where the level at which you could get by calling routines from the ROM ends. The rest of it was just

## Ones and zeroes

Data on the tape was written as a sequence of pulses. The processor took care of generating of these pulses, including the proper timing.

Each entry was preceded by a so-called pilot tone. It was created by a rectangular signal (where ON and OFF states were regularly alternated), which means that MIC output in the ON state was 2168 T long (T is the processor clock), and in the OFF state, 2168 T long as well. Boot tone prepared input circuits (if there were any) in a tape recorder to the correct volume level. This boot tone before the header lasted 5 seconds and before the data block two seconds. The duration of the phase tone was determined by a flag byte, all values less than 128 meant a &#8220;long&#8221; phase tone and more than 128 meant a short one.

After the pilot tone a sync pulse was generated. Its purpose was to let the loading routine know that the pilot tone ends and to begin to process the data. Sync pulse was 667 T ON and 735 T OFF long.

The sync pulse was then followed by actual bytes of data. First was the flag byte, then the contents, and eventually checksum. Bytes were sent bit by bit from the highest one. Every bit was represented by a pulse (state switching to ON and OFF), but differed in length. For logical 0 it was 855T in the ON state and 855T in the OFF state, for logical 1 it was doubled, thus 1710T and 1710T, respectively.

When reading data (which is what we are after here) processor then measured the time between each change of the input signal (ie. time between transitions, or the so-called &#8220;Edges&#8221;). The recording was therefore not sensitive to polarity (such as for aforementioned PMD in the first version), however the processor did not have much time to do anything else, since it had to listen to EAR input, count loops until the signal changed, and check for SPACE key being pressed (which interrupted the loading, as you surely remember).

## Assembler time

Now this is the moment to extract the ROM contents (_taken from &#8216;The Complete Spectrum ROM Disassembly&#8217; by Dr. Ian Logan and Dr. Frank O&#8217;Hara, as published by Melbourne House in 1983, [available at WOS](https://www.worldofspectrum.org/ldbytes.html)_):

<pre class="lang:asm decode:true ">THE 'LD-BYTES' SUBROUTINE
This subroutine is called to LOAD the header information (from 076E) and later
LOAD, or VERIFY, an actual block of data (from 0802).

0556 LD-BYTES   INC  D                  This resets the zero flag (D cannot
                                        hold +FF)
                EX   AF,AF'             The A register holds +00 for a header
                                        and +FF for a block of data
                                        The carry flag is reset for VERIFYing
                                        and set for LOADing
                DEC  D                  Restore D to its original value
                DI                      The maskable interrupt is now disabled
                LD   A,+0F              The border is made WHITE
                OUT  (+FE),A
                LD   HL,+053F           Pre-load the machine stack with the
                PUSH HL                 address - SA/LD-RET
                IN   A,(+FE)            Make an initial read of port 254
                RRA                     Rotate the byte obtained
                AND  +20                but keep only the EAR bit
                OR   +02                Signal RED border
                LD   C,A                Store the value in the C register
                                        (+22 for 'off' and +02 for 'on' - the
                                        present EAR state)
                CP   A                  Set the zero flag

The first stage of reading a tape involves showing that a pulsing signal
actually exists. (i.e. 'On/off' or 'off/on' edges.)

056B LD-BREAK   RET  NZ                 Return if the BREAK key is being pressed
056C LD-START   CALL 05E7,LD-EDGE-1     Return with the carry flag reset if
                JR   NC,056B,LD-BREAK   there is no 'edge' within approx.
                                        14,000 T states. But if an 'edge' is
                                        found the border will go CYAN

The next stage involves waiting a while and then showing that the signal is
still pulsing.

                LD   HL,+0415           The length of this waiting period will
0574 LD-WAIT    DJNZ 0574,LD-WAIT       be almost one second in duration.
                DEC  HL
                LD   A,H
                OR   L
                JR   NZ,0574,LD-WAIT
                CALL 05E3,LD-EDGE-2     Continue only if two edges are found
                JR   NC,056B,LD-BREAK   within the allowed time period.

Now accept only a 'leader signal'.

0580 LD-LEADER  LD   B,+9C              The timing constant
                CALL 05E3,LD-EDGE-2     Continue only if two edges are found
                JR   NC,056B,LD-BREAK   within the allowed time period
                LD   A,+C6              However the edges must have been found
                CP   B                  within about 3,000 T states of each
                JR   NC,056C,LD-START   other
                INC  H                  Count the pair of edges in the H
                JR   NZ,0580,LD-LEADER  register until 256 pairs have been found

After the leader come the 'off' and 'on' parts of the sync pulse.

058F LD-SYNC    LD   B,+C9              The timing constant
                CALL 05E7,LD-EDGE-1     Every edge is considered until two edges
                JR   NC,056B,LD-BREAK   are found close together - these will be
                LD   A,B                the start and finishing edges of the
                CP   +D4                'off' sync pulse
                JR   NC,058F,LD-SYNC
                CALL 05E7,LD-EDGE-1     The finishing edge of the 'on' pulse
                RET  NC                 must exist
                                        (Return carry flag reset)

The bytes of the header or the program/data block can now be LOADed or VERIFied.
But the first byte is the flag byte.

                LD   A,C                The border colours from now on will be
                XOR  +03                BLUE & YELLOW
                LD   C,A
                LD   H,+00              Initialize the 'parity matching' byte
                                        to zero
                LD   B,+B0              Set the timing constant for the flag
                                        byte.
                JR   05C8,LD-MARKER     Jump forward into the byte LOADing loop

The byte LOADing loop is used to fetch the bytes one at a time. The flag byte is
first. This is followed by the data bytes and the last byte is the 'parity'
byte.

05A9 LD-LOOP    EX   AF,AF'             Fetch the flags
                JR   NZ,05B3,LD-FLAG    Jump forward only when handling the
                                        first byte
                JR   NC,05BD,LD-VERIFY  Jump forward is VERIFYing a tape
                LD   (IX+00),L          Make the actual LOAD when required
                JR   05C2,LD-NEXT       Jump forward to LOAD the next byte
05B3 LD-FLAG    RL   C                  Keep the carry flag in a safe place
                                        temporarily
                XOR  L                  Return now if the flag byte does not
                RET  NZ                 match the first byte on the tape
                                        (Carry flag reset)
                LD   A,C                Restore the carry flag now
                RRA
                LD   C,A
                INC  DE                 Increase the counter to compensate for
                JR   05C4,LD-DEC        its decrease after the jump

If a data block is being verified then the freshly loaded byte is tested against
the original byte.

05BD LD-VERIFY  LD   A,(IX+00)          Fetch the original byte
                XOR  L                  Match it against the new byte
                RET  NZ                 Return if 'no match' (Carry flag reset)

A new byte can now be collected from the tape.

05C2 LD-NEXT    INC  IX                 Increase the 'destination'
05C4 LD-DEC     DEC  DE                 Decrease the 'counter'
                EX   AF,AF'             Save the flags
                LD   B,+B2              Set the timing constant
05C8 LD-MARKER  LD   L,+01              Clear the 'object' register apart from
                                        a 'marker' bit

The 'LD-8-BITS' loop is used to build up a byte in the L register.

05CA LD-8-BITS  CALL 05E3,LD-EDGE-2     Find the length of the 'off' and 'on'
                                        pulses of the next bit
                RET  NC                 Return if the time period is exceeded
                                        (Carry flag reset)
                LD   A,+CB              Compare the length against approx.
                CP   B                  2,400 T states; resetting the carry flag
                                        for a '0' and setting it for a '1'
                RL   L                  Include the new bit in the L register
                LD   B,+B0              Set the timing constant for the next bit
                JP   NC,05CA,LD-8-BITS  Jump back whilst there are still bits to
                                        be fetched

The 'parity matching' byte has to be updated with each new byte.

                LD   A,H                Fetch the 'parity matching' byte and
                XOR  L                  include the new byte
                LD   H,A                Save it once again

Passes round the loop are made until the 'counter' reaches zero. At that point
the 'parity matching' byte should be holding zero.

                LD   A,D                Make a furter pass if the DE register
                OR   E                  pair does not hold zero
                JR   NZ,05A9,LD-LOOP
                LD   A,H                Fetch the 'parity matching' byte
                CP   +01                Return with the carry flag set if the
                RET                     value is zero (Carry flag reset if in
                                        error)


THE 'LD-EDGE-2' and 'LD-EDGE-1' SUBROUTINES
These two subroutines form the most important part of the LOAD/VERIFY operation.
The subroutines are entered with a timing constant in the B register, and the
previous border colour and 'edge-type' in the C register.
The subroutines return with the carry flag set if the required number of 'edges'
have been found in the time allowed; and the change to the value in the B
register shows just how long it took to find the 'edge(s)'.
The carry flag will be reset if there is an error. The zero flag then signals
'BREAK pressed' by being reset, or 'time-up' by being set.
The entry point LD-EDGE-2 is used when the length of a complete pulse is
required and LD-EDGE-1 is used to find the time before the next 'edge'.

05E3 LD-EDGE-2  CALL 05E7,LD-EDGE-1     In effect call LD-EDGE-1 twice;
                RET  NC                 returning in between in there is an
                                        error.
05E7 LD-EDGE-1  LD   A,+16              Wait 358 T states before entering the
05E9 LD-DELAY   DEC  A                  sampling loop
                JR   NZ,05E9,LD-DELAY
                AND  A

The sampling loop is now entered. The value in the B register is incremented for
each pass; 'time-up' is given when B reaches zero.

05ED LD-SAMPLE  INC  B                  Count each pass
                RET  Z                  Return carry reset & zero set if
                                        'time-up'.
                LD   A,+7F              Read from port +7FFE
                IN   A,(+FE)            i.e. BREAK and EAR
                RRA                     Shift the byte
                RET  NC                 Return carry reset & zero reset if BREAK
                                        was pressed
                XOR  C                  Now test the byte against the 'last
                AND  +20                edge-type'
                JR   Z,05ED,LD-SAMPLE   Jump back unless it has changed

A new 'edge' has been found within the time period allowed for the search.
So change the border colour and set the carry flag.

                LD   A,C                Change the 'last edge-type' and border
                CPL                     colour
                LD   C,A
                AND  +07                Keep only the border colour
                OR   +08                Signal 'MIC off'
                OUT  (+FE),A            Change the border colour (RED/CYAN or
                                        BLUE/YELLOW)
                SCF                     Signal the successful search before
                RET                     returning


Note: The LD-EDGE-1 subroutine takes 464 T states, plus an additional 59 T
states for each unsuccessful pass around the sampling loop.
For example, therefore, when awaiting the sync pulse (see LD-SYNC at 058F)
allowance is made for ten additional passes through the sampling loop.
The search is thereby for the next edge to be found within, roughly, 1,100 T
states (464 + 10 * 59 overhead).
This will prove successful for the sync 'off' pulse that comes after the long
'leader pulses'.</pre>

In this article there is no space for detailed description of how the algorithm works, but let&#8217;s quickly go over some facts.

Routine disables interrupts. This is logical, since it is dependent on exact timing. That&#8217;s also the reason why loaders stored in slow RAM ($4000- $7FFF) will not work.

The last state of EAR input is stored in the C register, but it is shifted by 1 bit to the right, and the lowest three bits contains border color. For the leader it is 02 (red) / 05 (cyan), for data it is 01 (blue) / 06 (yellow). Why the right shift? During loading, a sequence LD A, $7F and IN A, ($FE) is processed. The value $7F is sent to the upper 8 bits of the address, therefore when reading a keyboard input it selects a line of keys B &#8211; N &#8211; M &#8211; Symbol Shift &#8211; Space. The status of these keys is in the lowest five bits (0-4), bit 6 contains an EAR state. Using the RRC instruction the lowest bit (status of the SPACE key) moves to the flag CY and EAR state moves to the 5th bit position. Therefore, if CY is zero, it means that a SPACE was pressed and loading ends.

The routine begins by detecting a signal that corresponds to the pilot tone (LD-LEADER). If 255 of such pulses were read, it was then believed that this was a pilot tone and the routine waits for a shorter synchronization pulse. When it arrives (LD-SYNC), you can retrieve data.

LD-MARKER reads 1 byte to register L. It begins with a value of 01, to serve as a counter. Gradually it fills with bits from the right by instruction RL L, the highest bits then passes into CY. If CY = 0, they keep loading further, but once CY = 1, it means that a complete set of eight bits was retrieved.

Key routines are LD-EDGE-n (wherein n is 1 or 2). LD-EDGE-1 first waits for a certain period of time (465T) and then determines whether the value of the EAR input has changed and compares it against the last stored value (in register C, see above). If it has not changed, the loop is repeated. For each loop the value in the registry B is increased. Once it gets to zero, it means &#8220;timeout&#8221; &#8211; the edge did not come in the expected time limit.

If the edge is found, the content of the register C is negated. This results both in a change of stored EAR value, but also in a change of the color border.

LD-EDGE-2 actually performs two LD-EDGE-1 in a sequence.

LD-EDGE output is as follows:

  * CY = 0, Z = 1 &#8211; during a time interval EAR change did not come (&#8220;timeout&#8221;)
  * CY = 0, Z = 0 &#8211; SPACE was pressed (BREAK)
  * CY = 1 &#8211; edge was found, the current value of the counter is in B

<span style="line-height: 1.5;">The counter in B counts, as I wrote, upwards. During every loop pass, which takes 58 processor ticks, B is incremented by one. For example, when reading a bit, routine LD-EDGE-2 is called, thus seeking two edges. The counter in B is set to $B0. This means that the timeout comes after $4F cycles ($FF- $B0). This represents a 2 x 465T of wait loop + 79 * 58T = 5512T. After all this time, the routine reports a signal failure.</span>

The resulting counter value is compared to $CB. If it is smaller, it is then evaluated as two short pulses of log. 0, if it is larger, then log. 1. The value of $CB means that the loop was done 27 times ($CB-$B0), specificaly that the two edges came at a time of less than 2496T. Let&#8217;s recall: For log. 0 the last two pulses take 1710T, for log. 1 they take 3420T. Therefore the difference between these two times is 2565 and this is roughly what I came up with. It is a bit less because of overhead (subroutine calls, evaluation etc., See LD-8-BITS).

## Loader hacking

This was not so difficult, was it? So now, let&#8217;s make some of those tricks &#8230;

First, reading routine is not completely &#8220;T-pedantic&#8221;, so few Ts here or there do not pose any major problem. If you want a simple effect, we can add it without any complicated adjustments.

### Border effects

We can for example change color of the stripes, if we do not like the default two-tone ones. How about a rainbow? Simply rewrite the end of the routine LD-EDGE:

<pre class="lang:asm decode:true">LD      A,C 
          INC     A 
          XOR     $20 
          AND     $27 
          LD      C,A 
          AND     $07 
          OR      $08 
          OUT     ($FE),A 
          SCF     
          RET</pre>

What&#8217;s going on here? Instead of negating of the contents of register C, we increase its value by 1 and negate the value of the fifth bit. Thanks to masking of the value of $27, we avoid the overflow which would affect the EAR bit. Therefore it will vary in the range 0-7 and create a rainbow effect border.

**Note:** If you intend to try this, be sure to place the loading routine in the upper 32 Kbytes of RAM!

If we add a pair of instructions XOR A; OUT ($FE), A to the end and before the instruction SCF, stripes in the border will change into short lines on a black background.

Here we can put any effects that affect a color border either by amending it, or by using it. What about the effect that Busy used with loader for Song In Lines 3 (rounded corners), it looks impressive, huh?



Yet it is not very hard&#8230; Four squares in the corners contain a simple pattern (&#8220;rounding&#8221;). Pixels that are equal to 1 (the color INK) will look as if they were part of the border and will show streaks. I did not examine how Busy does it, but I&#8217;d bet that in principle it&#8217;s done somehow like this:

<pre class="lang:asm decode:true">LD      A,C ;Change of the last "edge type"
          CPL     ;as well as BORDER color.
          LD      C,A 
          AND     $07 ;Taking just only the BORDER color
          OR      $08 ;MIC off
          OUT     ($FE),A ; Change border color
          OR      $30 ; A contains: 0 0 1 1 1 b b b (b = border color)
                  ; so PAPER=7, INK=border color, 
                  ; BRIGHT 0, FLASH 0
          LD      ($5800),A ;left top corner attribute
          LD      ($581F),A ;right top corner
          LD      ($5AE0),A ;left bottom
          LD      ($5AFF),A ;right bottom

          SCF     ;Set CY=1 as a "success"
          RET     ;before return</pre>

Border effects are mostly simple and fast enough, so we can squash them here and not worry about the timing too much. Usually they fit within tolerance.

### Simple effects with loaded content

During the loading we can certainly manage simpler operation with loaded data, either at the bit or byte level. Digisynth demos loader was able to perform real-time data extracting using Huffman decompression algorithm (Huffman suits this purpose quite well, you just need to have a decompression tree saved and pass through it according to the loaded bit). For those interested, I have prepared a reconstruction of the loader. But we will have a look at another case, and this will be a well known Mad Load &#8211; a routine that loads square images in a certain order. The video shows its improved version.



Mad Load used a very simple data format. After the flag byte, data for each square followed. Each took 11 bytes &#8211; lower and higher byte of a screen address where the square should be stored, then 8 bytes of video memory and a 1 byte of attribute. This was followed by another square &#8230;

This loading routine was slightly modified by Frantisek Fuka ([FUXOFT](https://www.worldofspectrum.org/infoseekpub.cgi?regexp=^Fuxoft$)) &#8211; he made LD-8-BITS into a subroutine which reads 1 byte to register L. This subroutine is then used in another subroutine to retrieve one of the squares (MAD_SQUARE). Squares loading subroutine is then called over and over again until there is data to be loaded from the tape, and when there isn&#8217;t, it stops and returns back. There is no checksum performed or anything.

Here is the Mad Loader source code. I only commented on the parts that differ from the standard code.

<pre class="lang:asm decode:true ">LD      HL,MAD_RETURN 
          PUSH    HL 
          JP      MAD_LOAD 
          NOP     
          NOP     
          NOP     
          NOP     
MAD_RETURN:       
          EI      
          RET     

MAD_LOAD:         
          DI      
          IN      A,($FE) 
          RRA     
          AND     $20 
          LD      C,A 
          CP      A 
LD_BREAK:         
          RET     NZ 
LD_START:         
          CALL    LD_EDGE_1 
          JR      NC,LD_BREAK 
          LD      HL,$0415 
LD_WAIT:          
          DJNZ    LD_WAIT 
          DEC     HL 
          LD      A,H 
          OR      L 
          JR      NZ,LD_WAIT 
          CALL    LD_EDGE_2 
          JR      NC,LD_BREAK 

LD_LEADER:        
          LD      B,$9C 
          CALL    LD_EDGE_2 
          JR      NC,LD_BREAK 
          LD      A,$C6 
          CP      B 
          JR      NC,LD_START 
          INC     H 
          JR      NZ,LD_LEADER 

LD_SYNC:  LD      B,$C9 
          CALL    LD_EDGE_1 
          JR      NC,LD_BREAK 
          LD      A,B 
          CP      $D4 
          JR      NC,LD_SYNC 
          CALL    LD_EDGE_1 
          RET     NC 

                  ; Mad Load itself
          CALL    LD_ONE_BYTE ; The first one is a flag byte. Drop it!
MAD_LOOP:         
          CALL    MAD_SQUARE 
          RET     NC 
          JR      MAD_LOOP 

MAD_SQUARE:       
          CALL    LD_ONE_BYTE ; Lower byte of address
          LD      A,L 
          EX      AF,AF' ; save into AF'
          CALL    LD_ONE_BYTE ; Upper address byte
          RET     NC 
          EX      AF,AF' 
          LD      H,L ; to the H register
          LD      L,A ; and the first one to the L, so I have a full address in HL
          LD      B,$08 ;8 bitmap bytes for each square
MAD_SCRN:         
          PUSH    HL ;save the address
          PUSH    BC ;and the counter
          CALL    LD_ONE_BYTE ; read 1 byte
          POP     BC ; restore the counter
          LD      A,L ; byte to accumulator
          POP     HL ; restore address for this byte
          RET     NC ; If some error occured, return
          LD      (HL),A ;Else store the byte into screen memory
          INC     H ; addr + 256 - it means "next screen microline"
          DJNZ    MAD_SCRN ; repeat for all 8 bytes
          LD      A,H ; Convert address from screen memory to attribute memory
          SUB     $08 ; First, sub 8 to get the original value
          RRA     
          RRA     
          RRA     ; H div 8
          AND     $03 ; lowest 2 bits
          OR      $58 ;$58, $59 or $5a - attribute memory address
          LD      H,A ; So now I have an attribute address in HL
          PUSH    HL ; save it
          CALL    LD_ONE_BYTE ; read one byte
          LD      A,L ; save it to accumulator
          POP     HL ; restore the address
          LD      (HL),A ; and put the attribute byte to a proper place
          RET     ; Finished, one square is done



LD_ONE_BYTE:      
          LD      B,$B2 
          LD      L,$01 
LD_8_BITS:        
          CALL    LD_EDGE_2 
          RET     NC 
          LD      A,$CB 
          CP      B 
          RL      L 
          LD      B,$B0 
          JR      NC,LD_8_BITS 
          SCF     
          RET     

                  ;-----------------------------------

LD_EDGE_2:        
          CALL    LD_EDGE_1 
FF78:     RET     NC 
LD_EDGE_1:        
          LD      A,$16 
LD_DELAY:         
          DEC     A 
FF7C:     JR      NZ,LD_DELAY 
FF7E:     AND     A 
LD_SAMPLE:        
          INC     B 
          RET     Z 
          LD      A,$7F 
          IN      A,($FE) 
          RRA     
          XOR     C 
          AND     $20 
          JR      Z,LD_SAMPLE 
          LD      A,C 
          INC     A 
          XOR     $20 
          AND     $27 
          LD      C,A 
          NOP     
          NOP     
          AND     $07 
          OR      $08 
          OUT     ($FE),A 
          SCF     
          RET     
</pre>

&#8230; And at this point I would to finish for now.

[Next time](https://www.uelectronics.info/2015/03/24/zx-spectrum-and-loaders-part-two/) we take a look at more complex effects, for example various counters of bytes or time remainging and other horseplay, for which we need more processor ticks, and so we will have to adjust the timing loop and chop our algorithms so that they fit into the time we have available. Meanwhile, please accept this brief &#8220;introductory to the mystery of loaders&#8221;, try to experiment yourself, and if you create some nice loader, show it off!

<small>(Translated by <a href="https://plus.google.com/+Ond%C5%99ejFicek/posts">Ondřej Ficek</a>)</small>