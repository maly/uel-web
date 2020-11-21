---
id: 2273
title: 'ASM80: New features in April'
date: 2018-04-05T10:04:57+01:00
author: Martin Maly
layout: post
guid: http://www.uelectronics.info/?p=2273
permalink: /2018/04/05/asm80-new-features-in-april/
categories:
  - My projects
  - Programming
tags:
  - ASM80
---
Dear ASM80 users&#8230;

I am still working on bug fixes and new features. Big THANKS to all who report me bugs or suggest new features (Jeff, Sandor etc.)

Bugfixes contain:

  * fixed error for macros without parameters
  * better error messages (some of them was misleading, like the &#8220;internal error&#8221;)
  * Z80 fix for IX+&#8230;, IY+&#8230; &#8211; now assembler checks limits (before was &#8220;modulo 256&#8221;)
  * Workspace downloading
  * Better listing format (for long labels etc.)
  * Better documentation of math operators

New features:

  * Did you try the [ASM TOOLS](https://www.asm80.com/tools/index.html)? It contains math function table generator and HEX 2 BIN converter!
  * ASM80 has a feature for download binary file (BIN) instead of HEX. Two new directives were added &#8211; binfrom and binto &#8211; for easy setting memory range to export. [Consult docs for further details](https://maly.gitbooks.io/asm80/directives.html).
  * IDE has now a &#8220;Find A Label Definition&#8221; feature. Just press CTRL (META) key and double click on the label name, e.g. in the CALL instruction, and editor will find the label definition.
  * IDE has now an auto-complete feature: write first two, three letters and press Ctrl+SPACE (on macOS it is Control too, not Meta &#8211; I am sorry, but the editor is a 3rd side software), the editor will suggest you symbol names.
  * Workspaces and its synchronization caused some problems. Big apologies to all who challenges data loss, I am really sorry about that. Opening workspace again now does not overwrite local content, but merge it together. Conflict files are stored in both variant, you have to decide which one is recent. There is a &#8220;workspace auto-sync&#8221; feature, which saves the whole workspace remotely on each file save. It can be a little slower, but your files are saved safely.

About workspaces and its syncing there are some scenarios you can face to:

  1. Open remote workspace A with the same workspace opened locally: Remote files are merged with local.
  2. Open remote workspace A with another workspace B opened locally: You can save local workspace B to remote storage (prompted) and then remote workspace A is opened.
  3. Press SYNC WORKSPACE: Current workspace is merged with remote version and then saved.
  4. Press SAVE WORKSPACE: A remote version of current workspace is OVERWRITTEN by local version! (Be caution, this is a scenario you can delete a file in remote storage!)
  5. How to delete a file in the remote workspace? Just delete the file locally and then SAVE workspace (not SYNC, because of synchronization will download the deleted file from the remote storage first)

I hope you find these updates useful.