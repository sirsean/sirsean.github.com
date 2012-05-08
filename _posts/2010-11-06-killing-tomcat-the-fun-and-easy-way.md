---
layout: post
title: Killing Tomcat, the fun and easy way!
tags:
- Java
- Ruby
status: publish
type: post
published: true
meta:
  _edit_last: '2'
  _wp_old_slug: ''
author: sirsean
---
For the longest time, I've gone through an annoying manual step of killing Tomcat when it needs to die.

    ps aux | grep tomcat

_Search for the right one (ie, not the grep), and manually enter the process id into:_

    kill -9 <process id>

Normally, Tedium is a good enough motivator for me to script something like this. But this time, I waited until Ridicule threw its hat into the ring. After a coworker asked why I didn't script it, it was really only a matter of time.

    #!/usr/bin/env ruby

    ps = `ps aux | grep catalina.startup.Bootstrap | grep -v grep`
    chunks = ps.split(" ")
    if not chunks.empty?
        pid = chunks[1]
        puts "Killing pid: #{pid}"
        `kill -9 #{pid}`
    end

And now all I have to do is:

    ~/code/killer/kill_tomcat.rb

There you go. Tomcat is dead.
