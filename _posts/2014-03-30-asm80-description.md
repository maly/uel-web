---
id: 331
title: ASM80 description
date: 2014-03-30T08:59:36+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=331
permalink: /2014/03/30/asm80-description/
categories:
  - Microprocessors
  - My projects
tags:
  - ASM80
---
My [online assembler ASM80](http://www.asm80.com) can assemble 6809 source code now. So let me show you its capabilities a little bit deeply.

<!--more-->

## CPU selection

CPU type is determined by file suffix. &#8220;.a80&#8221; means 8080 or 8085, &#8220;.z80&#8221; means Z80, &#8220;a65&#8221; means 6502 and &#8220;a09&#8221; means 6809.

## Basic operations

Write your code. Don&#8217;t forget to save it with some name. F9 or Compile compiles this code and makes filename.hex and filename.lst files.

## Source code format.

Line can begin with a label. Label should be followed by &#8220;:&#8221;, but it can be omitted.

Everything after a ; in a line is a comment (unlees the ; is part of a string literal, of course). There are no multiline comments.

String literals are written to the object file without any character set translation. In case you use punctuated character, the lower byte of its unicode representation will be used.

Blanks are significative only in string literals and when they separate lexical elements. Any number of blanks has the same meaning as one. A blank between operators and operands is allowed but no required except when the same character has other meaning as prefix (&#8216;$&#8217; and &#8216;%&#8217;, for example).

### Literals

Numeric literals can be written in decimal, binary, octal and hexadecimal formats. Several formats are accepted to obtain compatibility with the source format of several assemblers.

A literal that begins with $ is a hexadecimal constant, except if the literal is only the $ symbol.

A literal that begins with % is a binary constant, except if the literal is only the % symbol, in that case is an operator.

A literal that begins with a decimal digit can be a decimal, binary, octal or hexadecimal. If the digit is 0 and the following character is an X, the number is hexadecimal. If not, the suffix of the literal is examined: D means decimal, B binary, H hexadcimal and O or Q octal, in any other case is taken as decimal. Take care, FFFFh for example is not an hexadecimal constant, is an identifier, to write it with the suffix notation you must do it as 0FFFFh.

#### String literals.

There is one format of string literals. They should be single or double quote delimited. Both of these forms are equivalent.

A string literal of length 1 can be used as a numeric constant with the numeric value of the character contained. This allows expressions such as &#8216;A&#8217; + 80h to be evaluated as expected.

Identifiers are the names used for labels, EQU symbols and macro names and parameters. The names of the CPU mnemonics, registers and flag names, and of assemble directives are reserved and can not be used as names of identifiers. Reserved names are case insensitive, even if case sensitive mode is used.

Identifiers are not case sensitive.

### Expressions

You can use common mathematics expressions instead simple literal. You can use all operators like +, -, *, /, %, <<, >> etc. as well as the parenthesis.

## Directives

| Directive                     | Meanings                                                                                                                                                                                                                                                                                                                                                 |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| db (aliases: defb, fcb)       | Define Byte. The argument is a comma separated list of string literals or numeric expressions. The string literals are inserted in the object code, and the result of the numeric expression is inserted as a single byte, truncating it if needed. You can use DUP for entering N same values: DB 10 DUP (123) means &#8220;10 times value 123&#8221;   |
| dw (aliases: defw, fdb)       | Define Word. The argument is a comma separated list of numeric expressions. Each numeric expression is evaluated as a two byte word and the result inserted in the proper &#8220;endianity&#8221;. You can use DUP for entering N same values: DW 10 DUP (123) means &#8220;10 times value 123&#8221;                                                    |
| ds (aliases: defm, defs, rmb) | Define Space. Take one argument, which is the amount of space to define, in bytes.                                                                                                                                                                                                                                                                       |
| fill                          | Fill memory with a value. Take two arguments, the first is a value, the second is length of filled block (byte count).                                                                                                                                                                                                                                   |
| bsz (alias: zmb)              | Fill memory with a given count of zeros.                                                                                                                                                                                                                                                                                                                 |
| org _addr_                    | ORiGin. Establishes the origin position where to place generated code. Several ORG directives can be used in the same program, but if the result is that code generated overwrites previous, the result is undefined.                                                                                                                                    |
| .ent _addr_                   | ENTer point for debugging. I.e. **.ent $**                                                                                                                                                                                                                                                                                                               |
| .align _N_                    | The .align directive causes the next data generated to be aligned modulo N bytes.                                                                                                                                                                                                                                                                        |
| .phase _addr_                 | Continue to produce code and data for loading at the current address but assemble instructions and define labels as if they originated at the given address. Useful when producing code that will be copied to a different location before being executed.                                                                                               |
| .dephase                      | End phase block.                                                                                                                                                                                                                                                                                                                                         |
| .engine _name_                | Select type of engine (=virtual machine) for debugging. Types are: sbcz80, sbc65, sbc09, pmd85, pmi80&#8230;                                                                                                                                                                                                                                             |
| equ (alias: =)                | EQUate. Must be preceded by a label. The argument must be a numeric expression, the result is assigned to the label. I.e. **VIDRAM equ $4000**                                                                                                                                                                                                           |
| .if _cond_                    | Contional assembly. The argument must be a numeric expression, a result of 0 is considered as false, any other as true. If the argument is true the following code is assembled until the end of the IF section is encountered, else is ignored. The IF section is ended with a ENDIF directive. IF can&#8217;t be nested.                               |
| .ifn _cond_                   | IF NOT                                                                                                                                                                                                                                                                                                                                                   |
| .endif                        | End of the IF block                                                                                                                                                                                                                                                                                                                                      |
| .include _filename_           | Include a file. The file is readed and the result is the same as if the file were copied in the current file instead of the INCLUDE line. The file included may contain INCLUDE directives, and so on. INCLUDE directives are processed before the assembly phases, so the use of IF directives to conditionally include different files is not allowed. |
| .incbin _filename_            | Incbin stands for &#8220;include binary&#8221;. It allows any binary data to be included verbatim into the output. The argument is given in the same way as for include.                                                                                                                                                                                 |
| .macro _macro_name_           | Defines a macro, see [the chapter about macros](http://www.asm80.com/doc/help.html#macros).                                                                                                                                                                                                                                                              |
| .rept _počet_                 | Repeat a block of code substituing arguments. See [the chapter about macros](http://www.asm80.com/doc/help.html#macros).                                                                                                                                                                                                                                 |
| .endm                         | End of MACRO definition or REPT cycle.                                                                                                                                                                                                                                                                                                                   |
| .block                        | Start of logical block. All labels, defined in this block, are local. It means you can&#8217;t reference them from outside the block. If you want to define a label global, simply prefix it with &#8216;@&#8217;, like @LABEL:  
Good idea is to enclose INCLUDEd code into block.                                                                      |
| .endblock                     | End of BLOCK.                                                                                                                                                                                                                                                                                                                                            |

## Processor-specific syntax

### 6502

  * **Zero page** Assembler tries to determine if zero page mode is suitable. It needs the operand value is computable in the first pass (so no forward reference, no intensive math etc.) If you need implicitly select zero page mode, simply prepend asterisk (*) sign before the operand.

### 6809

  * **Direct, or extended?** Assembler tries to determine which mode is suitable. It needs the operand value is computable in the first pass (so no forward reference, no intensive math etc.) If you need implicitly select one mode, use signs < (for direct) or > (for extended) right before the operand.

### 6800

  * **LDA A or LDAA? **Use literally what you want to use. Compiler internally transfer all of these instructions into long syntax (without a space), so LDA A becomes LDAA etc.