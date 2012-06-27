---
layout: post
title: Information Density
tags: [Android, Mobile, MLB Scoreboard]
author: sirsean
---

I'm trying to sift through the complaints about the new MLB Scoreboard interface, to figure out which are knee jerk responses that don't need to be addressed, and which are on the money.

I tried to chase the idea of a "modern" design, by aping the slide-out menu that a lot of other apps have adopted; part of that was that the menu doesn't cover the entire screen. In other apps that seems to work better, and I think the reason is that people don't want to sit and stare at the menu, they want to use it to get where they're going.

But in MLB Scoreboard, one of the things people want to do is look at the schedule. And they have certain expectations about what that schedule is going to show them.

I think that was one of the big, actual problems with the new layout. The original schedule contained the full linescore of every game, the starting pitchers, the winning and losing and saving pitchers, the current inning and bases and outs situation, and the latest play.

But the new one only had the teams, the score, and the inning/base/out situation. It seemed like a big step backwards, because for what people were using the app for, it was.

![New slide-out menu](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-26-22-23-55.png "New slide-out menu")

I've added more of the linescore, including not only runs, but also hits and errors. And the latest play is there for the games that are in progress; before games start, the starting pitchers are there, and after games are over the winning and losing pitchers are displayed.

![New slide-out menu](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-27-11-34-41.png "New slide-out menu")

This is moving in the right direction, I think. Information density on the schedule screen has shot back up in the latest version. It still doesn't show the full inning-by-inning linescore, which I know will disappoint some people. But I continue to think I'm right that it's time to move away from that; it takes up _way_ too much space, and it didn't look very good. It used to be able to show 3-4 games on one screen, now it can show 8.

The schedule still slides out, but it now fills the whole screen. Again, having part of the screen essentially "wasted" was something that people didn't like, and I think it had to do with the information density issue. _Why are you leaving that space untouched and unused, and showing me so much less of the information I care about?!_

Oh, and one last thing. As part of my update, I jumped the **targetSDKVersion** up to the latest. That improves things, mostly, but it creates one huge problem. The menu button, where you can get to your settings and standings and such, [only appears on-screen on phones in ICS, not on tablets](http://stackoverflow.com/questions/8774317/the-missing-menu-button-in-honeycomb-and-ice-cream-sandwich). I mean, unless you use **ActionBar**, which of course is only available on Honeycomb and ICS ... which means 95% of my users can't use it.

When people complain about fragmentation, this is pretty much what they're talking about. So I had to do something kind of stupid, which you probably saw in that screenshot. There's now a menu button on the screen, within my app. I don't use the actual device menu button any more, which is probably going to be fine going forward since the menu button is going away. But I know it sucks for tens of thousands of my users* who have an older phone that has a menu button. I know there's a better way to do this, one that isn't quite so lame. I'll figure out what that is soon, I'm guessing.

_* Myself included._

Did I get it right? To find out, [go download MLB Scoreboard 2.2 now](https://play.google.com/store/apps/details?id=com.vikinghammer.mlb.scoreboard.full)!
