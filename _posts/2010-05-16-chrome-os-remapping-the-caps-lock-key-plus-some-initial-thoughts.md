---
layout: post
title: ! 'Chrome OS: Remapping the Caps Lock key, plus some initial thoughts'
tags:
- ChromeOS
- Linux
- Tinkering
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
This morning I installed Chromium OS on my old Acer Aspire One, which happens to be on its last legs and has been collecting dust in the bottom of a box in the back of my closet for the last several months. (The biggest two problems are 1: the AC adapter doesn't work consistently and sometimes won't charge the device, and 2: the OS options are slim, and slow, and Jolicloud was a huge disappointment.)

I used [the Chromium Flow build by Hexxeh](http://chromeos.hexxeh.net/), which is just a 300MB or so download and gives you a bootable USB drive you can use to check out Chromium OS -- or install to the computer's hard drive so you can use it without having a USB drive sticking out the side of your computer.

So far the main problem I've seen is that this thing supports Flash; that makes web pages load _painfully_ slow on a device with merely a 1.6 GHz Atom and 512 MB RAM. If Google wants Chrome OS to be a success, they're going to have to _really_ increase the minimum hardware requirements, or implement a very high quality version of "click to Flash" or something which blocks Flash from loading until the user explicitly asks for it. But this isn't about Flash, so I'll just leave it at that. Other than Flash, the Chromium OS experience is quite good.

With the tiny keyboard on the netbook, I was having quite a bit of trouble hitting the control keys; they're really small and tucked away down in the corner of the keyboard where my oversized, gnarled up fingers can't reach. So I hit **Ctrl-Alt-T** to open up a terminal window, which includes a cool window-sliding animation, and entered the following commands:

    xmodmap -e "remove lock = Caps_Lock"
    xmodmap -e "add control = Caps_Lock"

Then I switched back to the main browser window by hitting **Alt-Tab**, though hitting **F12** would give me an Expose-like panel view of all my open windows.

**Note:** The only way I've figured out to close terminal windows once they're open is to go back into them and issue the "exit" command. Chromium OS doesn't seem to have very robust window-management functionality, which isn't surprising given the way they'd expect you to use it.

So now the Caps Lock key is a Control button, and using the keyboard just got a whole lot more pleasant.

With some sexier hardware, a more comfortable keyboard, and a 3G connection, I can totally see myself using a Chrome OS device; that said, I can't see how they can ever make it as nice to use as an iPad.
