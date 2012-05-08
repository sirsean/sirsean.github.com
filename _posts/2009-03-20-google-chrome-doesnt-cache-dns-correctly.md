---
layout: post
title: Google Chrome Doesn't Cache DNS Correctly
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
A few days ago, Dreamhost moved me over to their new cluster, from the old server that had been having some problems. But this post isn't about Dreamhost; it's about Google Chrome and browsers.

After the switchover happened, visiting my sites worked just fine in Safari and Firefox; just go to the address and the browser loaded it up just fine.

But in Chrome, it was throwing errors. Address not found. Bad httpd_config. Nothing would go through. I left Chrome open (since there was an article I wanted to read open in one of the tabs), and the problem persisted for a few days. Finally, this morning, I restarted Chrome and it re-resolved the DNS and could actually find my sites.

So apparently Chrome doesn't use the OS-level DNS cache, and instead does it on its own. I understand that they think they can do a better job at networking than Windows XP (which I have to use at work), but in this case it puts itself at a significant disadvantage to all the other browsers.

I don't want to say that Chrome should just get rid of its DNS cache; they surely put significant time into it, and I'm sure they think the performance gains are significant, since they never have to go check DNS servers a second time. But it seems to me that they should be able to check whether or not someone has successfully visited a site during the current browser session (which can be long-lasting, sure, but they keep records of all of it), and if they have and when they try to visit it again and the cached DNS record doesn't find the site, that they should go out and check the DNS servers again to see if the site's address has changed.

Obviously it's rare for a website's IP address to change during a browser session. And most people, I suppose, don't leave their browser open all the time. But given that Chrome is clearly encouraging people to use websites as web applications, and therefore to leave the browser open all the time, I think they should cover this case.

I know it's probably just me, but this led to a couple of days worth of annoyance. And I'd rather not be subjected to that, even if the fix is copying the URLs of the articles I want to read and restarting the browser. I shouldn't have to do that.
