---
layout: post
title: Python/MySQLdb on Snow Leopard
tags:
- MySQL
- Python
- Snow Leopard
- Tinkering
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
This one is mostly for me, so I remember how I did this. I like developing webapps using [web.py](http://webpy.org), which obviously requires Python to be able to talk to MySQL if you want to, you know, store anything.

On my old laptop I ran into some difficulty getting that to happen -- for some reason OS X doesn't come with the necessary libraries for Python to talk to MySQL, which seems like an oversight to me -- so I just spun up an Ubuntu VM on which to do my web.py development. (sudo aptitude install mysqldb-python (or python-mysql, or something, whatever it actually is, do an aptitude search first) is pretty damn easy.)

But my new MBP came with Snow Leopard, and I wanted to get my development environment working without the VM. So ... first I installed the x86_64 version of MySQL (which says it's for Leopard, but it works fine in Snow Leopard). You also need to have XCode installed to compile MySQLdb.

I followed the directions [here](http://www.mangoorange.com/2008/08/01/installing-python-mysqldb-122-on-mac-os-x/) and [here](http://www.brambraakman.com/blog/comments/installing_mysql_python_mysqldb_on_snow_leopard_mac_os_x_106/), because neither one of them ended up working by themselves. My ~/.profile ended up looking like this:

	export PATH=$PATH:/usr/local/git/bin:/usr/local/mysql/bin

	export CC="gcc-4.0"
	export CXX="g++-4.0"

Note from the first link that step 4 appears to be unnecessary in the version of MySQLdb that I downloaded (1.2.3c1), which is newer than the one he used (1.2.2). The C code changes were already done.

I don't know if the symlink and setup_posix.py changes were important, but I did them anyway, because what's the big deal?

Anyway, now I don't have to bookmark those links.
