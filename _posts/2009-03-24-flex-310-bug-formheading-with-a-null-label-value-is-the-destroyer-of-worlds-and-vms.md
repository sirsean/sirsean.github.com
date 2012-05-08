---
layout: post
title: Flex 3.1.0 Bug, FormHeading With a Null Label Value is the Destroyer of Worlds
  (and VMs)
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
This afternoon I came across some rather annoying behavior in Flex 3.1.0, that I'm going to go ahead and call a bug.

I was using a FormHeading tag at the top of my form, and I wanted it to change dynamically. So I had something like this:
<pre>&lt;mx:Binding source="_object.title" destination="heading.label" /&gt;
...
&lt;mx:Form&gt;
	&lt;mx:FormHeading id="heading" /&gt;
&lt;/mx:Form&gt;</pre>
And it works just fine as long as _object exists and is not null. The value is filled in nicely. But once the component has been initialized and created, if at any point in the future _object is set to null*, then the Flex VM throws an exception and bombs out; in fact, no other code will even execute until you restart the application. That's worse than a usual Flex exception, which typically allow you to continue doing things even after the error occurs.

* And this happens frequently in this instance, because _object is bound to a field in my Cairngorm model locator, and on another page I set the bound value to null to reset its view.

But if instead of the FormHeading, I have this:
<pre>&lt;mx:Binding source="_object.title" destination="heading.text" /&gt;
...
&lt;mx:FormItem label="Title"&gt;
	&lt;mx:Label id="heading" /&gt;
&lt;/mx:FormItem&gt;</pre>
Then it works perfectly whether _object is set or is null. What the hell?

Well, let's take a look at the code in <a href="http://opensource.adobe.com/svn/opensource/flex/sdk/branches/3.1.0/frameworks/projects/framework/src/mx/containers/FormHeading.as">FormHeading.as</a> that's throwing the exception:
<pre>private function createLabel():void
{
	// See if we need to create our labelObj.
	if (_label.length &gt; 0)
	{</pre>
The exception is being thrown at the line accessing _label.length (_label is the string value of the text to display). Obviously, if _label is null then the exception gets thrown, because you can't do that.

So how come it works in <a href="http://opensource.adobe.com/svn/opensource/flex/sdk/branches/3.1.0/frameworks/projects/framework/src/mx/controls/Label.as">Label.as</a>?
<pre>public function set text(value:String):void
{
    // The text property can't be set to null, only to the empty string.
    if (!value)
        value = "";</pre>
This is exactly what FormHeading needs to do; check if the value is null, and instead of trying to access its length first, check to see if it's null. If it is, set it to an empty string, and <em>then</em> you can check its length.

Now, I haven't yet figured out an adequate workaround. Using a Label "works," but I don't think it's as elegant as FormHeading.  I really think a nullity check of _label should be added to FormHeading.createLabel(), so this would work.

I <em>suppose</em> this could be considered a "feature," if you want to say that a form's header/title mustn't be dynamic (which is false, I do it all the time, this is just the first time it's ever been possible for it to be null), or perhaps if a form <em>must</em> have a title (again false, not all forms have a FormHeading defined, and if the value is null it could just fall back to the non-FormHeading behavior of not showing anything).

So I don't buy that it's a feature. I think it's a bug, and I think it'd be easy to fix. (Given that the fix is as simple as adding if (!_label) { _label = ""; } at the beginning of the method.)

Unfortunately, my project can't wait for the next version of Flex, even assuming they decide to fix this. So I have to fix it with an inelegant, mostly unacceptable workaround.

While it's supremely annoying that this bug exists, it's nice that Flex is open source and I can investigate what the actual problem is. If I couldn't look at the source, I'd have to assume that the problem is in <em>my</em> code and fall back to an inelegant workaround (after trying for an inordinate amount of time to find a way to get it to work correctly) without knowing why.

So, Adobe, any chance you put that fix in soon so nobody else has to suffer this indignity? Thanks.
