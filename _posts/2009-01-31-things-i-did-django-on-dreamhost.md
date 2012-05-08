---
layout: post
title: Things I Did; Django on Dreamhost
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
A while back, I wrote an article that said <a href="http://vikinghammer.com/2009/01/10/damn-the-cloud/">goodbye to cloud computing</a>, at least for programs I write myself. I found that I couldn't trust the combination of AppJet and Facebook to work reliably enough to keep my application running, and I had no control to fix it if there were problems.

Well, the application I'd written for the AppJet/Facebook "platform" was a simple little thing called "Things I Did" -- the idea being to keep track of the things you've done over the course of a day, so that at the end of it you know what you did. I find I always have trouble remembering what I did at work when it comes time for the daily or weekly recap, so this would help me remember. And it'd be convenient to get a weekly report so I can even remember what I'd done for the week to help me update my status reports.

Well, I've rewritten the application such that I can host it myself. I prefer to write in Python, so naturally I went with a little Django application. The tough part was actually deploying it on Dreamhost. Fortunately, Jeff Croft has a great little tutorial on <a href="http://jeffcroft.com/blog/2006/may/11/django-dreamhost/">actually getting Django working on Dreamhost</a>. So after about an hour of setting it up, re-setting it up after something didn't work, and tweaking my application to work in the new environment, I've got the thing running. I really thing it should be easier than this to get Django applications running on Dreamhost -- and I don't know that you could have multiple Django applications running on one Dreamhost account. (You have to add a pointer to your application's settings.py in your .bash_profile ... which means to me that you can run a max of one application on one account, even though a single account can host many sites.) Ultimately, this is going to be a dealbreaker for me with Django on Dreamhost.

But, for now, I have <a href="http://things.vikinghammer.com/did/">Things I Did</a> up and running. I'll keep it up and running, until at some point in the future I rewrite it with something other than Django that's more suitable for Dreamhost, or move to another provider that's more amenable to Django. I definitely want to be able to write more than one application hosted on my account.

So check out <a href="http://things.vikinghammer.com/did/">Things I Did</a> if you have trouble remembering what you did yesterday!
