---
id: 380
title: 'AVR CDC &#8211; virtual COM port'
date: 2014-06-05T12:14:51+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=380
permalink: /2014/06/05/avr-cdc-virtual-com-port/
categories:
  - AVR
---
[AVR-CDC](http://www.recursion.jp/avrcdc/cdc-232.html) is low speed USB CDC (Communication Device Class) soft device, suitable for &#8220;a poor man&#8217;s FTDI replacement&#8221;. Its firmware is suitable even for ATtiny45. It can handle 1200-4800 Baud using internal oscillator on ATtiny45, but with ATtiny2313 or ATmegas can handle up to 38k.