---
layout: post
title: Things I Did Now Supports Things I Haven't Done Yet But Expect To Do At Some
  Point
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '2'
author: sirsean
---
Due to overwhelming demand from exactly 100% of our users, <a href="http://things.vikinghammer.com/">Things I Did</a> now has todo items.

A Thing can either be a thing you did, or a thing you have to do. If it's a todo, just enter an @ character as the first character when you enter it, followed by a space and then whatever else you want the Thing to be (including any number of tags).

Adding it in was pretty simple.
<pre>def checkForTodoItem(self):
    regex = re.compile('^@')

    if regex.match(self.text):
        self.text =  self.text.replace('@', '', 1).strip()
        return True
    else:
        return False</pre>
Note that I only want to grab 1 of the @, and only if the string starts with one. You can still have Things with @ characters in them. Just in case you wanted to do that. Which exactly 0% of our users have <em>ever</em> wanted to do. Hence, you must be crazy if you wanted to do that.

And then when I save it I just call
<pre>thing.todo = thing.checkForTodoItems()</pre>
and the deed is done.

I made it so the todo items show up separately from the regular things, at the top, and are a different color. The same is true even when you're viewing all the things marked with a tag, so you can tell which things you haven't done yet.

And you can mark an item done by clicking on the "Done" link on the main /did/ page. Marking it as no longer a todo item and just a Thing you did is as simple as:
<pre>if thing.todo:
    thing.todo = not thing.todo
    thing.save()</pre>
Bam. Done.

Things I Did just got a whole lot more useful. You don't have to wait until after you do a thing before you enter it in!
