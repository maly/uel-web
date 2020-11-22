---
id: 465
title: 'ASM80 news: Breakpoints'
date: 2015-01-17T21:32:13+01:00
author: Martin Maly
layout: post
guid: https://www.uelectronics.info/?p=465
permalink: /2015/01/17/asm80-news-breakpoints/
categories:
  - My projects
  - Uncategorized
tags:
  - ASM80
---
I&#8217;ve just uploaded brand new version of [ASM80 IDE](https://www.asm80.com). New build fixes bug in Z80 debugger and added new debug capability: Breakpoints.

<!--more-->

ASM80 debugger had limited capabilities &#8211; just &#8220;single step&#8221; and &#8220;50 steps&#8221; and Animate. With the new build, the &#8220;50 steps&#8221; is replaced with the &#8220;Step over&#8221; function. Step over works in the same way as Single step, with one difference: it runs the next instruction and stop on the instruction after it. So for common loads or math, it works exactly like Single step. The biggest difference is for calls. Step over execute the subroutine at full speed, and stop immediatelly after RET.

Debugger has a new function RUN. It works like Animate, but at full speed.

You can place breakpoints into your source code, simply by clicking on the line number. These breakpoints are persistent (i.e. you can close the editor, open it again, and those breakpoints stays at their places). When you emulate the code, executing (Animate or Run) stops on that lines.

With &#8220;Breakpoints&#8221; button you can manually add or delete breakpoint, as well as memory watchers. Memory watcher can stop the execution, when memory is read at given address, write to given address, or when one specific memory cell gets the specific value.

See all these functions in this short videotutorial (switch to HD for best experience)