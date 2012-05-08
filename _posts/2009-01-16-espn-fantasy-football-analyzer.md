---
layout: post
title: ESPN Fantasy Football Analyzer
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
I like to play fantasy football, but I think ESPN could offer more data.

Particularly, I want to know how bad I am at it, as well as a way to settle arguments about whose team is the best in the league. Typically, the team with the best record claims he's the best; however, everyone else notices that the player with the best record often has fewer points against than the other team. Also, some fantasy football players (and by some, I mean <em>all</em>) set their record improperly and players score big points on their bench.

If you have a bunch of players who score well for your bench, it means your team is good, just that <em>you</em> aren't very good at setting your roster.

So I wanted a way to calculate <em>optimum points</em> and <em>optimum record</em> to help settle these debates. The concept of optimum points is that it's your score if you'd set your starting roster optimally -- and started your players such that you get the most possible points. Your optimum record is what your record would be if you and all your opponents had played optimally every week. The team with the most optimum points is probably the best team; the team with the best optimum record is the one that should have won the league. Note that this is <em>not</em> necessarily the same team, given that there's still luck involved in who you play.

While the team with the fewest optimum points is the worst team in the league, the team with the largest gap between actual and optimum points has the worst owner. If your bench players keep scoring big, you're probably not very good at this.

So I came up with my <a href="http://github.com/sirsean/espn-fantasy-football-analyzer/tree/master">ESPN Fantasy Football Analyzer</a> to calculate this stuff. That's the GitHub repository, so go check it out and use it to analyze your fantasy team if you're curious about how bad you are at fantasy football.

I originally wanted to make it so it crawled ESPN's league page and analyzed all the games automatically -- the problem is that ESPN requires that you log in to see the league's schedule and boxscores, and I couldn't get Scrapy to successfully pass the login cookies to ESPN to authenticate and get the pages. So I just downloaded the quick boxscore from each game manually and put them into a directory structure that looks like this:
<pre>YEAR/
   WEEKS/
      GAMES</pre>
For example:
<pre>2008/
   1/
      1
      2
      3
   2/
      1
      2
      3</pre>
etc...

I go through each specified game, and use a series of regexes to pull out the data I need. I don't need to bore you with the details, but I needed team names, player names/ids, player positions, the "slot" the player played in that week, player points -- for all players on each team, whether they were in the starting lineup or the bench.

As I analyze each game, I keep track of the two teams involved, the score lines of each player on their team, and calculate the actual and optimum points for each team and come up with the actual and optimum winner. I add each GameScore to the Season.

Once I have all the GameScores, I analyze the entire season and pull out the teams and players from each game. It's at that point that I can get a definitive list of all the teams in the league and all the players who were on a roster at any point during the season.

I could try to explain the whole thing right now, but I think I'll either wait till another post or just skip it. I encourage you to give it a shot so you can finally know just how much better you should have done.

(It turns out my team wasn't as good as I thought -- my optimum record was actually worse than my actual record, and several teams had higher optimum points and larger gaps between actual and optimum.)

Oh, and another thing it does is record which players scored above their season average against your team. That's just so you know which players hate you.

For example, Deangelo Williams played me twice, and scored 32 and 34 points in those games. Deangelo Williams hates me and my fantasy football team.

Enjoy!
