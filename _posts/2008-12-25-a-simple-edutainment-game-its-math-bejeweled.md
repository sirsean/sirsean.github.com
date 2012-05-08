---
layout: post
title: ! 'A Simple Edutainment Game: It''s Math Bejeweled!'
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
I read an article earlier today, <a href="http://www.forbes.com/enterprisetech/2008/12/18/mitra-edutainment-entrepreneur-tech-enter-cx_sm_1219mitra.html">exhorting people to write "edutainment" software</a>; there were multiple complaints in the article: teaching is hard so teachers should be classroom managers who turn to an authoritative knowledge database for answers to their students' questions, children are addicted to video games so there should be video games that teach them things instead of waste their time, and emerging markets have less money so there should be software for cheap cell phones from which people can somehow learn.

I agree with the idea that there should be an authoritative knowledge database, so that teachers can teach the same curriculum nationwide. But solving the "some students already know what you're trying to teach them while others are unwilling/unable to learn the simplest material, and the former are the ones you should push while the later are the ones that require the most time and attention" problem is much more complicated than just some software, I think. And software for cheap cellphones that teaches physics to people in third world countries while they're working in rice paddies, while noble, is neither here nor there.
<blockquote>And that brings us to the humongous entrepreneurial opportunity that immersive, engaging educational software presents. Why is it that children are addicted to games like <em>World of Warcraft</em>? And why have we not yet created educational software and games that are equally addictive?</blockquote>
Some kind of MMORPG that teaches you math would sure be interesting ... except that those games are typically designed as a fantasy escape world, where the rules of reality don't apply. So those types of games wouldn't work that well for teaching, really. (History maybe. Not math or physics.) But I think the author was using World of Warcraft as an example, and that what she really meant was "any video game at all." In my opinion, Bejeweled would be a better example.

It's the kind of game people waste time with, and can jump in and out at their leisure. They can play for three minutes or three hours. If there were a version of Bejeweled that made you perform simple arithmetic rather than match colors together, that'd be the kind of game I could actually imagine some kid playing and learning from. (Adding numbers isn't much more difficult than matching colors, really.)

So this afternoon, after I got off work, I threw such a simple game together. I wrote it Flex, because that's what I've mainly been using at work, and it's pretty fun. Also, it's pretty much ideal for this sort of thing.

The idea is that there's a "target number" that you have to hit by adding numbers together from a grid. You click a number in the grid to select it, and keep selecting numbers until you've hit the target. You get points for the size of the selection (ie, the more numbers it takes to add up to the target, the more points you get), as well as the amount of time it takes to do it (the faster the better). The target number changes when you reach a certain amount of points.

The first thing I needed was a clickable Cell that shows a number and can either be selected or not. That's in Cell.as, and here's the important bits:

<pre>public function Cell(number:Number=1)
{
super();

this.number = number;

this.toggle = true;

this.setStyle("fontSize", 14);
this.width = 50;

this.addEventListener(MouseEvent.CLICK, _clickHandler);
}</pre>

That sets up the Cell and gets it ready to be clickable. (It extends LinkButton, so it's ready to be clicked. We set up how it looks and then connect an event listener to handle toggling between selected and unselected. Another important method:

<pre>public override function set selected(value:Boolean):void {
super.selected = value;

_selected = value;

if (this.selected) {
this.setStyle("color", "red");
this.setStyle("textRollOverColor", "red");
} else {
this.setStyle("color", "black");
this.setStyle("textRollOverColor", "black");
}
}</pre>

We need to override the "selected" setter to flip between the selected an unselected states -- for now I just have it so the number is red when it's selected and black when it isn't. It works well enough for me, but later on I'll have to figure something out to make it look better.

That's just the cells in the grid. The next thing we need is the grid itself. So I create NumberGridView.mxml and toss an &lt;mx:Grid /&gt; element in there. Then I want to dynamically build the grid full of Cells. I do that when the component has initialized:

<pre>private function _onInitialize():void {
_generateTargetNumber();

for (var i:int=0; i &lt; GRID_SIZE; i++) {
_cells.push([]);
var gridRow:GridRow = new GridRow();
grid.addChild(gridRow);

for (var j:int=0; j &lt; GRID_SIZE; j++) {
var cell:Cell = new Cell(_randomCellValue);
var gridItem:GridItem = new GridItem();
gridItem.addChild(cell);
gridRow.addChild(gridItem);

_cells[i][j] = cell;

cell.addEventListener(MouseEvent.CLICK, _cellClickHandler);
}
}

_resetTimer();
}</pre>

In that method, I generate a new target value, start the timer, and build the grid of cells. I first have to create a GridRow, then fill it with GridItems; each GridItem has a Cell inside it, and each Cell is seeded with a random number. Also, I add another event listener to each cell -- I want to handle some stuff at the grid level when a cell is clicked. Namely, I need to check if the target has been met and act accordingly:

<pre>private function _cellClickHandler(event:Event):void {
// get the sum of the selected cells
var sum:Number = _sumSelection;

// check if the target has been met
if (sum == _targetValue) {
// add points to the total based on the selection
_points += _calculatePoints();

// check if they can level up
if (_canLevelUp) {
_level += 1;

_generateTargetNumber();
}

// reset the start time
_resetTimer();

// generate new numbers for each of the cells
var cells:Array = _selectedCells;
for each (var cell:Cell in cells) {
cell.number = _randomCellValue;
cell.selected = false;
}
}
}</pre>

I just get the sum of the selected cells, and check if it matches the target. If it does, then I calculate how many points they got for that selection, see if they leveled up, generate new numbers for those selected cells, and reset the timer. No problemo.

When I want to get the currently selected cells, I need loop through my matrix of cells and use the public getter on the Cell object:

<pre>private function get _selectedCells():Array {
var cells:Array = [];
for (var i:int=0; i &lt; GRID_SIZE; i++) {
for (var j:int=0; j &lt; GRID_SIZE; j++) {
var cell:Cell = _cells[i][j];
if (cell.selected) {
cells.push(cell);
}
}
}

return cells;
}</pre>

And to generate new target values and cell values, I use the Math.random() method:

<pre>private function _generateTargetNumber():void {
var minTarget:Number = 5 * _level;
var maxTarget:Number = 10 * _level;
var random:Number = Math.random();
var target:Number = Math.round(random * (maxTarget - minTarget)) + minTarget;
_targetValue = target;
}</pre>

<pre>private function get _randomCellValue():Number {
var random:Number = Math.random();
var value:Number = Math.round(random * (_targetValue - 1)) + 1;
return value;
}
</pre>

The target value is based on your current level (the higher your level, the bigger the numbers get), and the cell values are based on the target value. Obviously, we don't want cell values to be higher than the target, that'd be pretty useless.

When I want to calculate the points, I use the current target value, the number of cells currently selected, and the amount of time for the selection.

<pre>private function _calculatePoints():Number {
var currentTime:Date = new Date();
var cells:Array = _selectedCells;

var seconds:Number = ((currentTime.getTime() - _startTime.getTime()) / 1000);

var points:Number = (cells.length * _level * 5);

points += _targetValue;

if (seconds &lt; 1) {
points *= 3;
} else if (seconds &lt; 2) {
points *= 2;
} else if (seconds &lt; 5) {
points *= 1.5;
} else if (seconds &gt; 10) {
points *= 0.33;
}

points = Math.round(points);

return points;
}</pre>

The faster you hit the target, the more points you get. But the most important thing is getting a long chain; that adds up points the fastest. The reason for that is that I don't want people just hitting the cell with the same target ... but at the same time if they try to chain together a bunch of 1's, it'll take a while and then they won't have any 1's left to fill out any selections when they need it.

One problem I have is the leveling up:

<pre>private function get _canLevelUp():Boolean {
if (_level == 1) {
return (_points &gt; 500);
} else if (_level == 2) {
return (_points &gt; 2500);
} else {
return false;
}
}</pre>

For now, obviously, I'm only handling two levels. I'd like to come up with a simple algorithm to define the level for any amount of points, rather than having to define the points for each level. Any ideas? Bear in mind that the number of points people can score increases linearly with their level (the points per selection increases with the target, but with higher targets it takes longer to hit the target).

Obviously, I didn't show all the code here. And it's still a work in progress. If you want to see all the code or contribute to it and help me out, <a href="http://github.com/sirsean/numbergrid/tree/master">go check it out at GitHub</a>. Or if you just want to play it and see what it's like, <a href="http://flexdemo.vikinghammer.com/NumberGrid/NumberGrid.html">go play</a>!

I do think these simple games are the ones that could get kids addicted and get them to play and possibly learn. I don't think the goal should be to try to get them to play some ridiculous MMORPG or shooter game where they somehow learn something. This post has gone on way too long, so I'll wrap it up. Hopefully someone found it useful in some way. And also hopefully my project and work gets wrapped up and released soon -- it's actually a large program, unlike this and most of my side projects. It gets fun when projects get really big.
