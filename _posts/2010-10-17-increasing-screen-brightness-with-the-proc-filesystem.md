---
layout: post
title: Increasing screen brightness with the /proc filesystem
tags:
- Fedora
- Linux
- Tinkering
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
I installed Fedora/LXDE on an old laptop this weekend, and it went well -- but when I told it to hibernate and then booted it back up, the screen was painfully dim. I wonder why that happened ... but since I'm not planning to go fixing any bugs in the hibernate code of either Fedora or LXDE, the _why it happened_ wasn't nearly as pressing a question as _what do you do about it_?

So I started digging around in the /proc filesystem, after not finding anything obviously related to screen brightness in 5 seconds of looking through the GUI tools. The **/proc/acpi/video/VGA/LCDD/brightness** file looked promising.

    [root@flower sirsean]# cat /proc/acpi/video/VGA/LCDD/brightness 
    levels:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
    current: 0

Ah ha, no wonder the screen was so dim! I just had to tell it to increase the brightness.

    [root@flower sirsean]# echo 5 > /proc/acpi/video/VGA/LCDD/brightness

The screen _immediately_ got much brighter, more usable. I was kind of surprised by that, to be honest, but I liked it.

    [root@flower sirsean]# cat /proc/acpi/video/VGA/LCDD/brightness 
    levels:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
    current: 5

Yup. I have a feeling I'll be needing to do this again at some point.
