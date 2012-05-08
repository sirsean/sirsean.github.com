---
layout: post
title: Damn the Cloud
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Hosting your applications in the cloud. It sounds so great. You just develop it, and then release it somewhere out in the internet, where it can run forever without you having to worry about it. It's cheaper, faster, and you don't have to maintain or pay for any systems. It can even scale rapidly to new users and usage as needed! Amazing!

Except that you cede control of your application to someone else. You're at their whim as to whether your application runs or not. If their systems have a problem, you're stuck. Not only are you unable to get your program running again, you're not even able to pull your code and data out and run it elsewhere -- you're locked into their system. You just have to trust them, that a) they will never raise their prices, b) they will never have an outage, and c) they will never go out of business and/or disappear. Do you trust <em>anyone</em> to meet those requirements?

As an example, I recently developed an application using AppJet, and released it on Facebook. Seemed like a good idea; it was just a simple application, and it was something I wanted to use myself and get out the door as quickly as possible without having to do any administrative rigamarole, like taking the time to set up Django on Dreamhost, or something.

Check out the <a href="http://appjet.com/app/892539283/source">code for the application</a>, or <a href="http://apps.facebook.com/thingsidid/">go use it at Facebook</a>. The basic idea is that you write down all the things you do over the course of the day, and then later you can go back and check what you did, to help you remember. I wanted it because at the end of the week my boss needs to update our project status and asks "What did you do this week?" And I can never remember. A week is a long time.

So it took me just an afternoon to create the application using AppJet's nice Javascript environment, and even less time to link it up to Facebook. It worked well enough, despite a couple of initial problems. First, their query language is pretty limited: for example, you can't use less-than or greater-than on dates (or anything else), so if I wanted to set up a date-based search to narrow down what I did by date, well, I can't. The other initial problem was that I didn't know you had to get the Facebook user to "allow" your application just to get their user id. The documentation said you needed to do that to get access to their personal information, but did not specify that the user id is considered personal information. I didn't discover that little bug until my application got its second user and I could see all their entries ... and discovered that they could see all of mine. It was the work of a moment to fix it, but it was a little embarrassing.

But once I got those initial problems ironed out, I had my application running "in the cloud." It was nice. I didn't have to worry about how it was running, because someone else was taking care of it. In fact, I <em>couldn't worry</em> about it, because I had absolutely no control. Which wouldn't have been a problem, except there soon started to be outages.

After a couple of weeks, Facebook started regularly having problems accessing AppJet. I'd log into AppJet to see what was up, and sometimes they'd be down, sometimes they'd be slow, sometimes they'd be working just fine and the inability to communicate just made no sense. The important thing is that there was nothing I could do about it.

When there's an outage on Dreamhost, I can shoot off an email to their support people asking what's going on, and I very quickly get a response explaining what the situation was, an estimate as to how long it'll take to fix it, and an apology for the problem. If I were paying them more money, I could probably get even better service.

But AppJet and Facebook are "free." While that means free of charge, they're also free of accountability, free of customer service.

It's become extremely irritating. The outages happen more than daily, and my application has become unusable. I'm back to having trouble remembering what I did at the end of the week.

So I'm going back to writing outside the cloud. I'll write Django apps and put them on my hosted account. I will not release applications on these cloud servers, as long as I have a choice. Hopefully people have a choice for a while longer.
