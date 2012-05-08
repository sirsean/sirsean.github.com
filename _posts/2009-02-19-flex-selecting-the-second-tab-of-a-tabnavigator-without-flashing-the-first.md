---
layout: post
title: ! 'Flex: Selecting the Second Tab of a TabNavigator Without Flashing the First'
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Flex's TabNavigator is a great thing. In normal situations, it's easy enough to use:

<code>&lt;mx:TabNavigator id="tabNavigator"&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;vh:ComponentZero id="componentZero" label="Zero" /&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;vh:ComponentOne id="componentOne" label="One" /&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;vh:ComponentTwo id="componentTwo" label="Two" /&gt;
&lt;/mx:TabNavigator&gt;</code>

It'll load up ComponentZero first, and there will be three tabs at the top of the TabNavigator that let you switch between them. Awesome. Except sometimes you find yourself in a situation where you want to load the <em>second</em> tab instead of the first, but you want the order to stay the same.

I was in that situation today. At first it seems like you can just do something simple:

<code>private function _onCreationComplete(event:Event):void {
&nbsp;&nbsp;&nbsp;&nbsp;tabNavigator.selectedChild = componentOne;
}</code>

That'll jump you over to the second tab almost right away. Problem solved, right? Wrong! When it builds the page, it flashes the first tab for a fraction of a second before it moves you over to the second tab. That doesn't look so great. And in MXML, there's really nothing you can do about it.

The only way I could figure out to get around this is to build out the children of the TabNavigator in ActionScript, building the second tab first, then inserting the first tab in front of it. It's kind of cumbersome and isn't as smooth as just doing it with MXML, but it gets you what you want. If, that is, you want what I want. Which you'd better.

<code>private var componentZero:ComponentZero;
private var componentOne:ComponentOne;
private var componentTwo:ComponentTwo;</code>

<code>private function _onCreationComplete(event:Event):void {
&nbsp;&nbsp;&nbsp;&nbsp;componentOne = new ComponentOne();
&nbsp;&nbsp;&nbsp;&nbsp;componentOne.label = "One";
&nbsp;&nbsp;&nbsp;&nbsp;tabNavigator.addChild(componentOne);</code>

<code>&nbsp;&nbsp;&nbsp;&nbsp;componentZero = new ComponentZero();
&nbsp;&nbsp;&nbsp;&nbsp;componentZero.label = "Zero";
&nbsp;&nbsp;&nbsp;&nbsp;tabNavigator.addChildAt(componentZero, 0);</code>

<code>&nbsp;&nbsp;&nbsp;&nbsp;componentTwo = new ComponentTwo();
&nbsp;&nbsp;&nbsp;&nbsp;componentTwo.label = "Two";
&nbsp;&nbsp;&nbsp;&nbsp;tabNavigator.addChild(componentTwo);
}</code>

<code>...</code>

<code>&lt;mx:TabNavigator id="tabNavigator" /&gt;</code>

Note that I had to create the second tab first and add it to the TabNavigator, then I had to use addChildAt() to add the tab that I want to appear in front of it.

If you're ever stuck in where I was, hopefully this helps. If you've got a better way to do this, I'm all ears.
