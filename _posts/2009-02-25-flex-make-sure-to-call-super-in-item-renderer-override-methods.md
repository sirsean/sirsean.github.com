---
layout: post
title: ! 'Flex: Make Sure to Call Super in Item Renderer Override Methods'
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Today I came across a bug where I was using an item renderer in a data grid and the row didn't highlight when you mouse over it and wouldn't get selected when you clicked on it (ie, moused over or clicked on the column, not the entire row). The item renderer consisted of a Text component (so I could use htmlText) inside of a VBox (so I could use horizontalScrollPolicy="off").

At first, it was tough to see what the problem could have been. The item renderer is very simple, and I'm using more complicated item renderers elsewhere that don't have this problem. I thought it might be a symptom of using Text, but switching it to a Label (and losing my HTML rendering) did not change anything.

That's when I noticed that my data setter override didn't look quite right:
<pre>public override function set data(value:Object):void {
    _text.htmlText = value.content;
}</pre>
Well, that's not enough! I was <em>this close</em> to trying to hack something together with mouseover and click event handlers, when I realized I needed to do this:
<pre>public override function set data(value:Object):void {
    super.data = value;
    _text.htmlText = value.content;
}</pre>
Apparently something in the super class's data setter does something vitally important, and you really shouldn't skip that. If you do, weird stuff can happen, including data grid columns that don't behave right on mouseover or click.

Lesson learned.
