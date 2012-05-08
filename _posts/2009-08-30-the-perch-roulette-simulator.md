---
layout: post
title: The Perch Roulette Simulator
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
So I was drinking off another demoralizing Twins loss on Saturday night, and bemusedly clicking around my Google Reader feeds which have somehow been growing and growing unchecked for the last few weeks. Having not read The Daily WTF for a while, I figured I'd try to clear some of them out. I came across one called [Knocking Me Off The Perch](http://thedailywtf.com/Articles/Knocking-Me-Off-The-Perch.aspx), which involved a programmer and a lawyer who walked into a casino,* and ended up inventing an algorithmic roulette strategy wherein they turned $10 into $400 simply by following a few easy steps.

_* Have you heard this one before?_

Well, here are the steps, as they described them:

> Then we realized something. We had just discovered The Perch, a roulette strategy that could not fail. The rules were defined as follows.

> 1. Perch someplace where you can see a number of roulette tables at once.
> 2. As soon as a table shows four consecutive blacks or reds, swoop in and place a bet.
> 3. If the bet is successful then return to the Perch. Otherwise, place another bet worth 150% of the previous bet (instead of $10, place $15).
> 4. If the second bet fails, then shrug your shoulders and return to the Perch.

The programmer ended up concluding, the next morning, that there is no strategy for winning at roulette. The lawyer maintained that there was, and that they'd discovered it.

None of this was particularly interesting, until the "Bring Your Own Code" section where they challenge you to write your own simulator to see just how rare it is to do what they did.

[So, I did.](http://github.com/sirsean/perch-roulette-simulator/tree/master)

According to my simulator, if you use this strategy you'll turn $10 into $400 a little over 50% of the time. Which doesn't sound bad, except that the games where you actually win and get your $400, you're standing there for over 330 spins of the roulette tables. Oh, and in the games you end up losing all your money? Well, those ones only last about 8 spins.

My first question is: If you have a 50% chance of increasing your money 40x over ... do you do it? (Even if it takes several hours?)

My second question is: Is there anything wrong with my simulator? I can't imagine that casinos would allow such a _huge_ arbitrage situation to exist on the floor.
