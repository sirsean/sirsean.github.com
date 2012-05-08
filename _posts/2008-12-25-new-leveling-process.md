---
layout: post
title: New Leveling Process
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
So this morning, I came up with a considerably better way to define the levels for my number grid game. (Yes, I did that on Christmas morning. What?)

Instead of the ridiculous if-statement I put together last night, which would have been more than a little cumbersome to maintain, I now have a simple list of point values that define the points required for each level. It's not the algorithm I was hoping for (I guess I have to do some more thinking), but at least I can add more levels simply.

Here's the array with the levels defined:
<pre>private static const LEVELS:Array = [
0,
200,
600,
1000,
1500,
2500,
3800,
5500,
];</pre>
The way it works, the index of the array is the level, and the value at each index is the points required to graduate from that level. Since you start at level 1, the first value in the array (index zero) is useless.

To check if it's time to level up, I've replaced the ridiculous if statement with this:
<pre>private function get _canLevelUp():Boolean {
return (_points &gt; LEVELS[_level]);
}</pre>
Awesome. I think that part, at least, is pretty nice.

As before, <a href="http://github.com/sirsean/numbergrid/tree/master">see the code at GitHub</a> or just go <a href="http://flexdemo.vikinghammer.com/NumberGrid2/NumberGrid.html">play the game here</a>.
