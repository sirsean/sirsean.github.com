---
layout: post
title: Using ActionBar
tags: [Android, Mobile, MLB Scoreboard]
author: sirsean
---

With MLB Scoreboard 2.0, I tried to "modernize" the interface with a slide-out menu. It didn't work, but today I'm trying to modernize it yet again. This time, I'm going with **ActionBar** as my attempt at making the interface newer and fancier; this is the direction Google wants you to go with your apps, many other app makers are doing it, and it unifies how the menu works between phones and tablets.

![New schedule](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-07-01-14-28-27.png "New schedule")

You can see that the old bar with the arrows (to go forward or backward one day at a time) has been replaced with a blue ActionBar at the top of the screen.

You can refresh the current data, or go forward or backward one day at a time with the buttons on the ActionBar, or use the "overflow" menu to access the rest of the features. If your phone has a physical menu button, that overflow menu icon doesn't appear; instead, the menu appears when you click your menu button.

Even though ActionBar was introduced with Android 3.0, and changed further with 4.0, Google _does_ provide a compatibility library that allows you to use it on previous versions of Android. I finally figured out how to get that to work, so MLB Scoreboard _still works_ on pre-Honeycomb devices.

This resolves the [menu junkiness](http://vikinghammer.com/2012/06/27/information-density/) I mentioned in my last post, so I think people will like it more.

You'll also note that the "R H E" has been moved to the top of the screen and only appears once, rather than in every single row. That saves a ton of wasted space. And the pitcher that got the save has been added back to the schedule, for those who care about that.

If this sounds like it's moving in the right direction, [go download MLB Scoreboard 2.3 now](https://play.google.com/store/apps/details?id=com.vikinghammer.mlb.scoreboard.full)!
