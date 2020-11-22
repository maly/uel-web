---
id: 300
title: Intel 8080 emulator in JavaScript
date: 2014-01-03T20:43:42+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=300
permalink: /2014/01/03/intel-8080-emulator-in-javascript/
categories:
  - Uncategorized
tags:
  - ASM80
---
[8080js](https://github.com/maly/8080js) is _Yet Another 8080 Emulator_. But why?

[Chris Double&#8217;s js8080 emulator](https://bluishcoder.co.nz/js8080/) is pretty good if you want to play around. But it has a really big issues &#8211; some instructions simply don&#8217;t work, some instructions set the flags totally bad and DAA instruction is probably guessing rather than computing. Stephan Tramm did a pretty good work with his [i8080 CP/M emulator](https://www.tramm.li/i8080/). But it is still slightly differrent from original (&#8220;silicon&#8221;) CPU.

The biggest challenge were two testing software: so-called &#8220;[Kelly Smith Test](https://github.com/begoon/i8080-core/blob/master/TEST.ASM)&#8221; and &#8211; the greatest one &#8211; [8080 Exerciser](https://www.idb.me.uk/sunhillow/8080.html) (a.k.a. Sunhillow). The first one does some basic tests, it checks known border states and simply says: OK, or Error at&#8230; The second one does a lot of combinations of data manipulating instructions, and the result is a set of CRC codes (one for each test case). They are compared with CRCs from real &#8220;silicon&#8221; CPUs. It checks all non-standard states very well and deeply.

My emulator passed Kelly Smith test a long time ago. But Exerciser was a really big issue. It takes a lot of changes and tests and fixes, but &#8211; it works!

So I keep my promise and release this JS emulator as open source library. Here it is: [8080js](https://github.com/maly/8080js)

PS: Big thanks to [Roman Borik](https://pmd85.borik.net/) for his help.