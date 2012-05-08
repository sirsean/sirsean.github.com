---
layout: post
title: ! 'Eclipse-free Java & Flex: MXML/Actionscript in MacVim'
tags:
- Flex
- Java
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
At work, I've been doing Flex & Java work pretty much exclusively for a good long while now, and the standard toolchain for that centers around Eclipse; that's what all of us use at work. But Eclipse is slow as molasses and uses a _ton_ of memory ... and tends to break down if you're running 15+ Tomcat projects and compiling/building several different Flex projects. Last week, Eclipse finally gave up; it crashed, and then it wouldn't start up again. I could have continued fighting with it in a misguided attempt to get back to "normal" -- ie, get back to a slow and painful workflow with repeated interruptions while I wait for Eclipse to finish doing whatever it's doing.

Instead, I decided to go for an Eclipse-free workflow.

I'm using Maven to compile my code, run my tests, and build my JAR, WAR and SWF files. MacVim is my editor (though regular old vim would work just fine). I keep a Finder window open to my workspace so I can easily see directory structures and open files. And I've written some scripts that take the Maven-built WAR files and publish them to the Tomcat webapps directory so I can run them.

I posted a while ago over at my old blog about [how to get syntax highlighting working in vim for MXML and Actionscript](http://seancode.blogspot.com/2008/01/flex-mxml-highlighting-in-vim.html); that solution still works well, but it's important to note that [vim's syntax files go into a different directory* when you're using MacVim](http://stackoverflow.com/questions/1107857/macvim-syntax-file-for-cs):

    /Applications/MacVim.app/Contents/Resources/vim/runtime/syntax/

_* You will have to right-click and select "Show Package Contents" to get there._

Here are the links to the files:

- [actionscript.vim](http://abdulqabiz.com/files/vim/actionscript.vim)
- [mxml.vim](http://abdulqabiz.com/files/vim/mxml.vim)

After putting the two files in the proper location, the following lines go into the ~/.vimrc file:

    au BufNewFile,BufRead *.mxml set filetype=mxml
    au BufNewFile,BufRead *.as set filetype=actionscript
    syntax on
    autocmd FileType mxml set smartindent
    autocmd FileType as set smartindent

Frankly, I think this gives vim _at least_ as much syntax-highlighting capability as Eclipse/FlexBuilder _ever_ had, and I don't actually find myself missing the code completion all that much. (If anyone knows of any good code completion solutions for MXML/AS beyond the simple ctrl-N ... let me know.)

So far, my Eclipse-free existence has been going brilliantly. I'll try to post a few more times, fleshing out a little bit more about the environment.
