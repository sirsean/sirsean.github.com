---
layout: post
title: MLB Scoreboard 2.0
tags: [Android, Mobile, MLB Scoreboard]
author: sirsean
---

[MLB Scoreboard](https://play.google.com/store/apps/details?id=com.vikinghammer.mlb.scoreboard.full) has been by far my most popular app; as of today, there are 52,828 active users. This is far, far more popular than I'd ever imagined it could get. I still vividly remember the day it passed 1000 users, and how that seemed like an impossibly large number. Now, I remember the (much more recent) day my _daily_ downloads dropped below 1000, and how bad that felt.

Why bring it up now? Because the app has gotten a little bit crufty over the last year. When I originally released it, the app was literally just a scoreboard -- it showed you the linescores of all the games, but you couldn't click through to get any detail. Over time, I added all the features that people seem to like: individual game detail pages, play by play, video highlights, boxscores, individual player stat pages, and a live scoring widget. What had been a very fast and simple app had gotten a little bit less fast, and quite a bit less simple. And the interface started to seem a little stale to me.

So today, I'm very happy to release [**MLB Scoreboard 2.0**](https://play.google.com/store/apps/details?id=com.vikinghammer.mlb.scoreboard.full), which simplifies some things and fancies up the interface.

![New slide-out menu](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-23-08-27-59.png "New slide-out menu")

The first thing you'll probably notice is the new slide-out menu system, where the daily schedule lives. You can still page back and forth through the days, and jump to a specific day; but now, the schedule isn't its own Activity, so when you choose a game the schedule just slides away until you bring it back. This, I think, makes things seem smoother and faster.

![Game info](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-23-08-28-20.png "Game info")

As you can see, if you've been a regular user of MLB Scoreboard, everything is still here, but is just a little bit different. The full linescore has been moved into the info tab (instead of always living above the tabs), which gives more space to all the other tabs. Instead of being white text on a black background, it's now black text on a white background, which is something a few people complained about; I think it looks better.

![Play by play](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-23-08-28-37.png "Play by play")

The play by play gets a big benefit from the extra space, and also seems more readable because of the new color scheme.

![Batting boxscore](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-23-08-28-46.png "Batting boxscore")

You can get the menu to slide back out either by clicking the logo in the top left corner of the screen (like how the "ActionBar" works on Android 3.0+, but this isn't actually an ActionBar because I still want it to work on Android 2.1, 2.2, and 2.3), or by clicking the back button when you're viewing an individual game.

![Video highlights](/wp-content/uploads/mlb-scoreboard/Screenshot_2012-06-23-08-29-09.png "Video highlights")

The video highlights are still here.

Hopefully everybody likes the new interface, and I get a bunch of new downloads before MLB takes heed and I [go the way of Batter's Box](http://langui.sh/2012/06/20/farewell-batters-box/).

So [get it while it's hot](https://play.google.com/store/apps/details?id=com.vikinghammer.mlb.scoreboard.full)!
