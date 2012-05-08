---
layout: post
title: Eclim Disappointment
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
For a while, I've been wishing that I could use a Vim editor inside Eclipse. I'm basically forced to use Eclipse at work, but editing text in Vim is just so much more pleasant. For many months, I've just lived with the Eclipse text editor. Until yesterday, when I was alerted to the existence of Eclim, which promises to allow me to use Vim as an editor within Eclipse, and claims to offer many interoperability features which allow me to get all the code-completion benefits of Eclipse and all the text editing benefits of Vim. Sounds like a perfect solution!

Well, I didn't want to potentially break my work environment, so I waited until I got home, and opened up my Windows XP machine (to more closely mimic my work laptop, which is also unfortunately a Windows machine). I installed GVim, which is apparently the style of the time in Windows land. I then opened up the Eclim installer and followed the rather simple directions.

It only took a few minutes, and I had the eclimd server running from within Eclipse. I grabbed a file and opened it up with Vim, and it opened right inside Eclipse ... exactly what I wanted!

Right?

Well ... no.

Unfortunately, at least on Windows, Vim uses the hideous system console font by default. You can change it ... but it doesn't save the new font between files, or even between instantiations of Vim. You have to change the font within Vim each time you open a file, even if you're opening a file you've previously opened and set the font. <em>Maybe</em> there's a way to permanently set the font ... but there doesn't appear to be an option for it. Either way, why would it be this difficult/impossible to change the default font, and why would said default font be the worst one this side of Comic Sans?

Secondly, the Vim editor requires focus before you can actually use it ... but Eclipse retains focus after opening a file. So after opening a file, you have to click in it in order to start working. Annoying. But that's just step one of the total lack of integration between Vim and Eclipse. The code completion does not work, and it doesn't automatically include imports (which was one of my main desired features for this). You can't switch editor tabs via the keyboard, and none of the Eclipse keybindings work from within the editor. You <em>can</em> muddy up your .vimrc to get this "working," but I don't necessarily want to do that to my.vimrc ... and .vimrc's nmap command doesn't appear to support multiple-control-key bindings (ie, Ctrl-Shift-O does not work, you have to come up with some other keystroke for that functionality ... not that that particular one would even work).

If you hit the big X on the tab to close the window, it'll throw up a null pointer exception for you before actually closing the editor tab. Which would be annoying, except that re-opening that file is even worse. It throws another null pointer exception, with <em>another</em> modal dialog behind it explaining that there's a Vim swap file for that file, giving you the option of editing anyway, restoring or deleting the swap file, etc. So you have to remember that not only can you not use Ctrl-W to close a tab (at all), you can't even close it via the GUI. You <em>must</em> use :q to close an editor tab.

Oh, and the "Outline" view doesn't work if you use the Vim editor; apparently that feature of Eclipse interacts with the Java editor, as does the constantly-compile-my-source-code feature -- it doesn't compile until you save the file.

<em>Maybe</em> this eclim plugin works better in OS X and Linux, where both Vim and interprocess communication are a little more natural than they are in Windows. But frankly, I don't feel like finding out. I have to use it at work anyway, where for some reason I don't have a Mac. And this experience has been annoying enough that I'm not even interested any more.

Eclim does not work, and is a huge disappointment.
