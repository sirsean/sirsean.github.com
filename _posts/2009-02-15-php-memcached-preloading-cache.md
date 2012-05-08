---
layout: post
title: PHP Memcached Preloading Cache
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
I'm working on a project for a high-read, low-write environment, and one of the things we want to do is keep a database as far away from the production webservers as possible.

So I want to generate content somewhere else before uploading it to the webservers to be displayed. But we tried just generating all the static content, and it's not going to work; too much duplicated data and wasted space, and too many separate files.

Instead, I made a flat file with all the necessary information in it, with each tuple contained to its own line and data fields separated by | characters. Given one of two keys, I can scan through the file and find the tuple it belongs to. Once I've gotten the data from the file, I stick them into the templates on the webserver as opposed to generating it all at once.

But it seems wasteful to scan the file <em>every time</em> I need to look up a tuple. This is a high-read environment, and I don't want to spend all our time spinning through a flat file looking up the tuples. So I need memcached. On Ubuntu, it's really easy to set it up:
<pre>sudo aptitude install php5-cli
sudo aptitude install php5-memcache
sudo aptitude install memcached
memcached -d -m 1024 -l 127.0.0.1 -p 11211
sudo /etc/init.d/apache2 restart</pre>
At this point you can run PHP from the command line, access memcached from it, and a memcached server is running and is only accessible from this server, and Apache has been restarted so it can load up the new PHP extension.

Since I upload the new datafile daily, I'm going to need to flush the cache and preload some tuples into the cache -- for our higher-volume customers it'd be nice if we <em>never</em> need to look them up based on a customer visit. This is why we needed to run PHP from the command line; we have a script that does this.
<pre>$memcache = new Memcache;
$memcache-&gt;connect('localhost', 11211);</pre>
Now that we've connected to the memcached server, we want to flush it out; because of a limitation in PHP's memcache plugin, we need to wait one second after flushing before entering anything new into the cache.
<pre>$memcache-&gt;flush();
$time = time() + 1;
while (time() &lt; $time) { /* spin */ }</pre>
Now we're ready to look up some tuples and cache them. Here's where we load the keys we want to look up:
<pre>$ini = parse_ini_file(PRELOAD_FILENAME);</pre>
What? Oh right ... here's the file with the codes we want to load:
<pre>codes[] = dffo1000
codes[] = dffo50000
codes[] = dffo12121
...</pre>
I'll need to generate this list of most-frequently-used codes on the server and upload it to the webserver every time I generate a new datafile, but it shouldn't ever get that big and it still beats the trash out of generating all the content statically.

In PHP's ini files, putting [] at the end of the variable name means it's an array. Now we can loop through these and get each one like so:
<pre>foreach ($ini['codes'] as $code) { /* lookup and cache! */ }</pre>
I don't want to get into the actual lookups right now, but the caching looks like it's working great. And that was really the number one goal here.
